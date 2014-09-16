---
layout: post
title: Using Grunt with Yii AssetManager
---

Yii framework has very simple and powerfull asset versioning system. Using AssetManager you can simply publish bunch of asset files for each module of Yii project independently. AssetManager generates version key for every published path. One thing is missing - the building tasks. This gap can be filled by Grunt task runner.

##Yii AssetManager basics

It is very easy to start using Yii AssetManager. Only a few lines of configuration is enough to make it work.

*In your config.php*

```php
...

'components' => array(
    'assetManager' => array(
        'basePath' => 'assets',
        'baseUrl' => '/assets',
    ),
),

...

```

Where *basePath* is directory where assets should be published and *baseUrl* is an url where published assets can be accessed.

Next you have to publish your assets...

```php
Yii::app()->assetManager->publish(
	Yii::getPathOfAlias('application.assets')
);
```

...and use it

```php
$assetsBase = Yii::app()->assetManager->getPublishedUrl(
	Yii::getPathOfAlias('application.assets')
);

$imgUrl = $assetsBase . '/img/layout/image.jpg';
```

Yii will create subdirectory in your *basePath* directory named with calculated assets version (based on published path and last modification time) and copy (or link) there files from published path.

##To publish or not to publish?

Most of tutorials recommend running AssetManager's *publish()* in *init()* method of application base controller so assets will be published at first request. It can be ok for small websites but for large amount of files, publishing can take some time and can be broken by connection timeout.

I recommend to create console command which publish our assets.

```php
class AssetsCommand extends CConsoleCommand {
	public function publish($path) {
		Yii::app()->assetManager->publish(
			Yii::getPathOfAlias($path)
		);
	}
}
```

Now you can simply publish assets before deploy using command

*./yiic assets publish --path=application.assets*

##Adding Grunt

We have working AssetManager and console command for publishing our assets. Let's try to use Grunt tasks to combine, uglify and publish our asset files.

First we have to split our *basePath* to *src* and *build* directories. The *src* will contain not processed files and the *build* directory will contain output of Grunt.

*Sample Gruntfile.js*

```javascript
module.exports = function(grunt) {

  grunt.initConfig({
    srcDir: 'assets/src',
    buildDir: 'assets/build',
    uglify: {
      dist: {
        files: {
          '<%= buildDir %>module1.min.js': [
          	'<%= srcDir %>/a.js',
          	'<%= srcDir %>/b.js',
          	'<%= srcDir %>/c.js'
          ]
        }
      }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-uglify');

  grunt.registerTask('default', ['uglify']);

};
```

The *grunt* command will create module1.min.js file in out assets/build directory. Only one step is missing - publish. To run shell yiic command, grunt-shell plugin can be used.


```javascript
module.exports = function(grunt) {

  grunt.initConfig({
    srcDir: 'assets/src',
    buildDir: 'assets/build',
    uglify: {
      dist: {
        files: {
          '<%= buildDir %>module1.min.js': [
          	'<%= srcDir %>/a.js',
          	'<%= srcDir %>/b.js',
          	'<%= srcDir %>/c.js'
          ]
        }
      }
    }
    shell: {
        publish: {
            command: './protected/yiic assets publish --path=<%= buildDir %>'
        }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-shell');

  grunt.registerTask('default', ['uglify', 'shell:publish']);

};
```

We have created building system for processing and versioning assets files using power of Yii AssetManager and Grunt task manager. Examples are not production ready but can be used as strong basis of production deploy system.