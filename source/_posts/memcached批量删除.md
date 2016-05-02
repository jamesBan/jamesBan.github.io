title: memcached批量删除
date: 2015-02-09 19:49:30
tags: memcached
---


都说Cache是万金油，哪里不舒服抹哪里。不过要想批量删除缓存可没那么简单，
memcached 不支持批量删除缓存键。

## 遍历方式

```
	
	<?php
	/**
	 * Function to get all memcache keys
	 * @author Manish Patel
	 * @Created:  28-May-2010
	 * @modified: 16-Jun-2011 
	 */
	function getMemcacheKeys() {
	
	    $memcache = new Memcache;
	    $memcache->connect('127.0.0.1', 11211) or die ("Could not connect to memcache server");
	
	    $list = array();
	    $allSlabs = $memcache->getExtendedStats('slabs');
	    $items = $memcache->getExtendedStats('items');
	    foreach($allSlabs as $server => $slabs) {
	        foreach($slabs AS $slabId => $slabMeta) {
	           $cdump = $memcache->getExtendedStats('cachedump',(int)$slabId);
	            foreach($cdump AS $keys => $arrVal) {
	                if (!is_array($arrVal)) continue;
	                foreach($arrVal AS $k => $v) {                   
	                    echo $k .'<br>';
	                }
	           }
	        }
	    }   
	}//EO getMemcacheKeys()
	?> 
```

代码摘自 php手册

## 版本号方式

通过维护一个版本号管理缓存键,删除缓存时直接更新版本号，则带有版本号的缓存到过期时间时自动被删除([memcached的删除机制和发展方向](http://blog.charlee.li/memcached-003/))。

```

	<?php
	
	$cache  = new Memcache();
	$cache->connect('127.0.0.1', 11211);
	
	$key1 = genKey('key1');
	$cache->set($key1, 'this is test1', 0, 60);
	
	$key2 = genKey('key2');
	$cache->set($key2, 'this is test2', 0, 60);
	
	
	print_r($cache->get($key1)); //this is test1
	
	print_r($cache->get($key2)); //this is test2
	
	//修改
	delVersionKey();
	
	//生成代码号的key
	function genKey($key) 
	{
		global $cache;
	
		$versionKey = getVersionKey();
		$version = $cache->get($versionKey);
	
		if($version === false) {
			$cache->set($versionKey, rand(1, 100));
		}
	
		return $key . $version;
	}
	
	//版本号前缀
	function getVersionKey()
	{
		return 'public_version_';
	}
	
	//更新本版号值
	function delVersionKey()
	{
		global $cache;
		$cache->increment(getVersionKey());
	}
	?>


```


## 打tag方式

存储时关联缓存键和tag(可以多个)。删除时通过tag找到管理的缓存键，便利删除。


```
	class KeyEnabled_Memcached extends Zend_Cache_Backend_Memcached
	{
	
	    private function getTagListId()
	    {
	        return "MyTagArrayCacheKey";
	    }
	    
	    private function getTags()
	    {
	        if(!$tags = $this->_memcache->get($this->getTagListId()))
	        {
	            $tags = array();
	        }
	        return $tags;
	    }
	    
	    private function saveTags($id, $tags)
	    {
	        // First get the tags
	        $siteTags = $this->getTags();
	        
	        foreach($tags as $tag)
	        {
	            $siteTags[$tag][] = $id;
	        }
	        $this->_memcache->set($this->getTagListId(), $siteTags);        
	    }
	    
	    private function getItemsByTag($tag)
	    {
	        $siteTags = $this->_memcache->get($this->getTagListId());
	        return isset($siteTags[$tag]) ? $siteTags[$tag] : false;
	    }
	
	    /**
	     * Save some string datas into a cache record
	     *
	     * Note : $data is always "string" (serialization is done by the
	     * core not by the backend)
	     *
	     * @param  string $data             Datas to cache
	     * @param  string $id               Cache id
	     * @param  array  $tags             Array of strings, the cache record will be tagged by each string entry
	     * @param  int    $specificLifetime If != false, set a specific lifetime for this cache record (null => infinite lifetime)
	     * @return boolean True if no problem
	     */
	    public function save($data, $id, $tags = array(), $specificLifetime = false)
	    {
	        $lifetime = $this->getLifetime($specificLifetime);
	        if ($this->_options['compression']) {
	            $flag = MEMCACHE_COMPRESSED;
	        } else {
	            $flag = 0;
	        }
	        $result = $this->_memcache->set($id, array($data, time()), $flag, $lifetime);
	        if (count($tags) > 0) {
	            $this->saveTags($id, $tags);
	        }
	        return $result;
	    }
	
	    /**
	     * Clean some cache records
	     *
	     * Available modes are :
	     * 'all' (default)  => remove all cache entries ($tags is not used)
	     * 'old'            => remove too old cache entries ($tags is not used)
	     * 'matchingTag'    => remove cache entries matching all given tags
	     *                     ($tags can be an array of strings or a single string)
	     * 'notMatchingTag' => remove cache entries not matching one of the given tags
	     *                     ($tags can be an array of strings or a single string)
	     *
	     * @param  string $mode Clean mode
	     * @param  array  $tags Array of tags
	     * @return boolean True if no problem
	     */
	    public function clean($mode = Zend_Cache::CLEANING_MODE_ALL, $tags = array())
	    {
	        if ($mode==Zend_Cache::CLEANING_MODE_ALL) {
	            return $this->_memcache->flush();
	        }
	        if ($mode==Zend_Cache::CLEANING_MODE_OLD) {
	            $this->_log("Zend_Cache_Backend_Memcached::clean() : CLEANING_MODE_OLD is unsupported by the Memcached backend");
	        }
	        if ($mode==Zend_Cache::CLEANING_MODE_MATCHING_TAG) {
	            $siteTags = $newTags = $this->getTags();
	            if(count($siteTags))
	            {
	                foreach($tags as $tag)
	                {
	                    if(isset($siteTags[$tag]))
	                    {
	                        foreach($siteTags[$tag] as $item)
	                        {
	                            // We call delete directly here because the ID in the cache is already specific for this site
	                            $this->_memcache->delete($item);
	                        }
	                        unset($newTags[$tag]);
	                    }
	                }
	                $this->_memcache->set($this->getTagListId(),$newTags);
	            }
	        }
	        if ($mode==Zend_Cache::CLEANING_MODE_NOT_MATCHING_TAG) {
	            $siteTags = $newTags = $this->getTags();
	            if(count($siteTags))
	            {
	                foreach($siteTags as $siteTag => $items)
	                {
	                    if(array_search($siteTag,$tags) === false)
	                    {
	                        foreach($items as $item)
	                        {
	                            $this->_memcache->delete($item);
	                        }
	                        unset($newTags[$siteTag]);
	                    }
	                }
	                $this->_memcache->set($this->getTagListId(),$newTags);
	            }
	        }
	    }
	}
```

代码摘自[zendframework Memcache should support tags](http://framework.zend.com/issues/browse/ZF-4253)

参考

- [memcached的伪命名空间的一种实现方式](http://iyuanbo.iteye.com/blog/1177919)

- [缓存方式扩展，缓存Tag(命名空间)的实现](http://huacnlee.com/blog/group-caches-with-tag-or-namespace/)
