title: yii-tips
date: 2015-02-14 19:22:53
tags: yii
---

- 默认控制器
    1. 普通控制器
    
            defaultController => 'site'
    
    2. module内的默认控制器    
    
            modules => array(
                'web' => array(
                    'class' => 'application.modules.web.WebModule',
                    'defaultController' => 'index'
                )
            )
    
    3. 默认控制器为module内的控制器
    
            defaultController => 'web/site'

- 别名

    1. 内置别名有
    
            Yii::setPathOfAlias('application',$this->getBasePath());
            Yii::setPathOfAlias('webroot',dirname($_SERVER['SCRIPT_FILENAME']));
            Yii::setPathOfAlias('ext',$this->getBasePath().DIRECTORY_SEPARATOR.'extensions');
    
    2. 获取别名的路径
    
            Yii::app->getAliasPath('webroot');

    3. 自定义别名

            Yii::setPathOfAlias('test',$this->getBasePath());

- 路由

    1. 后缀

            'urlSuffix'  => '.html'
    2. 忽略大小写
    
            'caseSensitive' => false,
    3. 隐藏module
        
            '/<controller:\w+>' => 'web/<controller>',
            '/<controller:\w+>/<action:\w+>' => 'web/<controller>/<action>',
            '<controller:\w+>/<action:\w+>/<id:\d+>'=>'web/<controller>/<action>'
- 日志

    1、 显示执行的``sql``

            'routes' => array(
                array(
                  'class' => 'CWebLogRoute',
                   'levels' => 'trace,warning,error,info',
                   'categories' => 'system.db.CDbCommand.*',
               )
            ),
    2、自定``category``调试控制器台程序

        array(
            'class'=>'CFileLogRoute',          
            'categories'=>'debug.*',
        ),
    使用

        Yii::log($message, 'info', 'debug');
- ActiveRecord

    1. 自动记录 创建时间、修改时间

            protected function beforeSave()
            {
                if ($this->isNewRecord) {
                    $this->add_time = time();
                } else {
                    $this->modify_time = time();
                }
                return parent::beforeSave();
            }



    2. 乐观锁(版本号)

            class PostModel extends CActiveRecord
            {
                public function rules()
                {
                    return array(
                        array('version', 'required'),
                        array('id', 'exists', 'on' => 'update', 'criteria' => array('condition' => 'version <'. $this->version))
                    );
                }
            }
            
            $model1 = PostModel::model()->findByPk(1);
            $model2 = PostModel::model()->findByPk(1);
            
            $model1->version = $model->version++;
            $model->save();//true
            
            
            $model2->save();//false

- 显示控制器``action``执行时间

        protected function afterAction($action)
        {
            $time = sprintf('%0.5f', Yii::getLogger()->getExecutionTime());
            $memory = round(memory_get_peak_usage() / (1024 * 1024), 2) . "MB";
        
            echo "Time: $time, memory: $memory";
            parent::afterAction($action);
        }

- 维护通知``main.php``

        return array(
         .... 
             'catchAllRequest'=>array(
                  'offline/notice'
                  'otherParams'=>'value',
              ),
         ....
        ),
