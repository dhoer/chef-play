# Play Cookbook

[![Cookbook Version](http://img.shields.io/cookbook/v/play.svg?style=flat-square)][cookbook]
[![Build Status](http://img.shields.io/travis/dhoer/chef-play.svg?style=flat-square)][travis]

[cookbook]: https://supermarket.chef.io/cookbooks/play
[travis]: https://travis-ci.org/dhoer/chef-play

Installs the [distribution artifact](https://www.playframework.com/documentation/2.5.x/Production) (2.2+),
created by the dist task, as a service.

It is recommended that you include a `application.conf.erb` template file within the distribution artifact to configure 
environment specific variables like application secret.  
 
To include the .erb file in your distribution artifact, copy `application.conf` file and paste it as 
`application.conf.erb` in the same directory. Then replace the environment specific values with variables. These
variable names must match variable name passed into `config_variables`.

For example, replace `play.crypto.secret = "changeme"` with `play.crypto.secret = "<%= @secret %>"` in 
`application.conf.erb` file, then pass the value as a parameter into `config_values` attribute of Play resource:
  
So if application.conf.erb contained:

```ruby
play.crypto.secret = "<%= @secret %>"
```

And Play resource was called with:

```ruby
play 'servicename' do
  source 'https://example.com/dist/myapp-1.0.0.zip'
  config_variables(
    secret: 'abcdefghijk'
  )
  action :install
end
```

This would result in creating or overriding application.conf as:

```ruby
play.crypto.secret = "abcdefghijk"
```

Note that application configuration template can be external from distribution artifact.  See `config_template` 
attribute below for more information.

## Requirements

- Java (must be installed outside this cookbook)
- Chef 11+

### Platforms

- Centos/RedHat
- Ubuntu 

### Cookbooks

- ark

## Usage

See [play_test](https://github.com/dhoer/chef-play/tree/master/test/fixtures/cookbooks/play_test) cookbook
for an example using play cookbook to install distribution artifact as a service.

### Attributes

* `servicename` - Service name to run as.  Default value: the `name` of the resource block
* `source` - URL to archive or directory path to exploded archive. 
* `checksum` - The SHA-256 checksum of the file. Use to prevent the resource from re-downloading a file. 
When the local file matches the checksum, the chef-client will not download it.
* `project_name` - Used to identify start script executable.  Defaults to project name derived from standalone 
distribution filename, if project_name not provided.
* `version` - Version of application.  Defaults to version derived from standalone distribution filename, if version 
not provided. Not needed if source is a directory.
* `user` - User to run service as.  Default value: `play`
* `args` - Array of additional configuration arguments.  Default value: empty array `[]` 
* `config_variables` - Hash of application configuration variables required by application.conf.erb template.  
Default value: empty hash `{}`
* `config_template` - Path to configuration template.  Path can be relative, or if the template file is outside dist 
path, absolute.  Default value: `conf/application.conf.erb`
* `config_path` - Path to application configuration file. Path can be relative, or if the config file is outside 
standalone distribution, absolute. This is also the path to the location in which a file will be created and the name 
of the file to be managed by template. Default value: `conf/application.conf`
* `pid_dir` - The pid directory. Default value: `/var/run/play`

### Examples

To `install` a standalone distribution as service from remote archive:

```ruby
play 'servicename' do
  source 'https://example.com/dist/myapp-1.0.0.zip'
  config_variables(
    foo: 'bar'
  )
  args([
    '-Dhttp.port=8080',
    '-J-Xms128M',
    '-J-Xmx512m',
    '-J-server'
  ])
  action :install
end
```

To `install` a standalone distribution as service from local file:

```ruby
play 'servicename' do
  source 'file:///var/chef/cache/myapp-1.0.0.zip'
  config_variables(
    foo: 'bar'
  )
  args([
    '-Dhttp.port=8080',
    '-J-Xms128m',
    '-J-Xmx512m',
    '-J-server'
  ])
  action :install
end
```

To `install` a standalone distribution as service from exploded archive:

```ruby
play 'sample_service' do
  source '/var/local/mysample'
  project_name 'sample'
  config_variables(
    foo: 'bar'
  )
  args([
    '-Dhttp.port=8080',
    '-J-Xms128m',
    '-J-Xmx512m',
    '-J-server'
  ])
  action :install
end
```

### Override Attributes

You can also use override attributes in the environment file (the attribute to be overridden must 
not be set in play LWRP; otherwise, override attribute will be ignored):

```ruby
{
  "override_attributes": {
    "play": {
      "config_variables": {
        "foo": "bar"
      },
      "args": [
        "-Dhttp.port=8080",
        "-J-Xms128m",
        "-J-Xmx512m",
        "-J-server"
      ]
    },
    ...
  },
  ...
}
```

## ChefSpec Matchers

This cookbook includes custom [ChefSpec](https://github.com/sethvargo/chefspec) matchers you can use to test your 
own cookbooks.

Example Matcher Usage

```ruby
expect(chef_run).to install_play('servicename').with(
  source: 'https://github.com/dhoer/play-java-sample/releases/download/1.0/play-java-sample-1.0.zip',
  config_variables: {
    secret: 'abcdefghijk'
  }
)
```
      
Cookbook Matchers

- install_play(name)

## Getting Help

- Ask specific questions on [Stack Overflow](http://stackoverflow.com/questions/tagged/chef-play).
- Report bugs and discuss potential features in [Github issues](https://github.com/dhoer/chef-play/issues).

## Contributing

Please refer to [CONTRIBUTING](https://github.com/dhoer/chef-play/blob/master/CONTRIBUTING.md).

## License

MIT - see the accompanying [LICENSE](https://github.com/dhoer/chef-play/blob/master/LICENSE.md) file for 
details.