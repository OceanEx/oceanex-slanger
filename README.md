# OceanEx Slanger

The OceanEx Slanger inherits from the unmaintained Slanger. The project is backed by OceanEx dev team.

We will do regularly bug fixes and security updates. 

We might future add more features or provide performance improvements for OceanEx Slanger. The maintenance will continue until we find better option. 

Feel free to log any issue or contribute to the code base.

**Important! OceanEx Slanger is not supposed to be included in your Gemfile. RubyGems is used as a distribution mechanism. If you include it in your app, you will likely get dependency conflicts. 
PRs updating dependencies for compatibility with your app will be closed!**

OceanEx Slanger is a standalone server ruby implementation of the Pusher protocol.  It is not designed to run inside a Rails or sinatra app, but it can be easily installed as a gem.

# How to use it

## Requirements

- Ruby 2.6.3 or greater
- Redis

## Server setup

Most linux distributions have by defualt a very low open files limit. In order to sustain more than 1024 ( default ) connections, you need to apply the following changes to your system:
Add to `/etc/sysctl.conf`:
```
fs.file-max = 50000
```
Add to `/etc/security/limits.conf`:
```
* hard nofile 50000
* soft nofile 50000
* hard nproc 50000
* soft nproc 50000
```

## Cluster load-balancing setup with Haproxy

If you want to run multiple slanger instances in a cluster, one option will be to balance the connections with Haproxy.
A basic config can be found in the folder `examples`.
Haproxy can be also used for SSL termination, leaving slanger to not have to deal with SSL checks and so on, making it lighter.

## Installation instruction

The OceanEx Slanger depends on ruby 2.6.3 and above, please install ruby 2.6.3 first before install the OceanEx Slanger.
It could also run perfectly on the latest 2.7.1. If you want to align with the latest ruby, you might clone the source
code and compile yourself. 

### Linux(Ubuntu)

You could install the right version of ruby via rbenv

```
sudo apt-get install rbenv
rbenv install 2.6.3
rbenv global 2.6.3
```

Usually, install the Oceanex Slanger should be pretty easy with the following command. OceanEx Slanger is just a variation of the unmaintained Slanger.
The execution kept the same way as before. Please check our release note to see what is new in additional to the unmaintained Slanger. 

```
gem install oceanex-slanger
```

### Mac

Install the ruby version via brew

```
brew install ruby
```

You might also install via rbenv 
```
brew install rbenv
rbenv install 2.6.3
rbenv global 2.6.3
```

Installation should be pretty straightforward on linux, however, install OceanEx Slanger might fail on mac os.
This is due to the c compiler converts the warning to error when building the native extension.

If you see installation fails due to `implicit-function-declaration`, you could try the following step to suppress the warning.

```
gem install oceanex-slanger -- --with-cflags="-Wno-error=implicit-function-declaration"
```

## Start the OceanEx Slanger in local environment

Both the app key and app secret are just random string, you could choose any string. However, it is recommended to be long
enough to keep secure.

Oceanex slanger also depends on redis service, specify the redis url when launching the OceanEx Slanger. 

```
slanger --app_key $APP_KEY --secret $APP_SECRET -r $REDIS_URL
```

If all went to plan you should see the following output to STDOUT

```

    .d8888b.  888
   d88P  Y88b 888
   Y88b.      888
    "Y888b.   888  8888b.  88888b.   .d88b.   .d88b.  888d888
       "Y88b. 888     "88b 888 "88b d88P"88b d8P  Y8b 888P"
         "888 888 .d888888 888  888 888  888 88888888 888
   Y88b  d88P 888 888  888 888  888 Y88b 888 Y8b.     888
    "Y8888P"  888 "Y888888 888  888  "Y88888  "Y8888  888
                                         888
                                    Y8b d88P
                                    "Y88P"


Slanger API server listening on port 4567
Slanger WebSocket server listening on port 8080
```

## Start the OceanEx Slanger in Docker environment 

The OceanEx slanger supports running in docker environment and such approach is already encapsulated in the make command.
The dependent Redis docker image is also automatically downloaded and started.

For the app key and app secret, please check `docker-compose.yaml` and modify as needed.

### Build the docker image
```
make build
```

### Start the OceanEx slanger
```
make up
```

### Stop the service
```
make down
```

## Modifying your application code to use the Slanger service

Once you have a Slanger instance listening for incoming connections you need to alter you application code to use the Slanger endpoint instead of Pusher. Fortunately this is very simple, unobtrusive, easily reversable, and very painless.


First you will need to add code to your server side component that publishes events to the Pusher HTTP REST API, usually this means telling the Pusher client to use a different host and port, e.g. consider this Ruby example

```ruby
...

Pusher.host   = 'slanger.example.com'
Pusher.port   = 4567
```

You will also need to do the same to the Pusher JavaScript client in your client side JavaScript, e.g

```html
<script type="text/javascript">
  var pusher = new Pusher('#{Pusher.key}', {
    wsHost: "0.0.0.0",
    wsPort: "8080",
    wssPort: "8080",
    enabledTransports: ['ws', 'flash']
  });
</script>
```

Of course you could proxy all requests to `ws.example.com` to port 8080 of your Slanger node and `api.example.com` to port 4567 of your Slanger node for example, that way you would only need to set the host property of the Pusher client.

# Configuration Options

Slanger supports several configuration options, which can be supplied as command line arguments at invocation. You can also supply a yaml file containing config options. If you use the config file in combination with other configuration options, the values passed on the command line will win. Allows running multiple instances with only a few differences easy.

```
-k or --app_key This is the Pusher app key you want to use. This is a required argument on command line or in optional config file

-s or --secret This is your Pusher secret. This is a required argument on command line or in optional config file

-C or --config_file Path to Yaml file that can contain all or some of the configuration options, including required arguments

-r or --redis_address An address where there is a Redis server running. This is an optional argument and defaults to redis://127.0.0.1:6379/0

-a or --api_host This is the address that Slanger will bind the HTTP REST API part of the service to. This is an optional argument and defaults to 0.0.0.0:4567

-w or --websocket_host This is the address that Slanger will bind the WebSocket part of the service to. This is an optional argument and defaults to 0.0.0.0:8080

-i or --require Require an additional file before starting Slanger to tune it to your needs. This is an optional argument

-p or --private_key_file Private key file for SSL support. This argument is optional, if given, SSL will be enabled

-b or --webhook_url URL for webhooks. This argument is optional, if given webhook callbacks will be made http://pusher.com/docs/webhooks

-c or --cert_file Certificate file for SSL support. This argument is optional, if given, SSL will be enabled

-v or --[no-]verbose This makes Slanger run verbosely, meaning WebSocket frames will be echoed to STDOUT. Useful for debugging

--pid_file  The path to a file you want slanger to write it's PID to. Optional.
```

# Why use Slanger instead of Pusher?

There a few reasons you might want to use Slanger instead of Pusher, e.g.

- You operate in a heavily regulated industry and are worried about sending data to 3rd parties, and it is an organisational requirement that you own your own infrastructure.
- You might be travelling on an airplane without internet connectivity as I am right now. Airplane rides are very good times to get a lot done, unfortunately external services are also usually unreachable. Remove internet connectivity as a dependency of your development envirionment by running a local Slanger instance in development and Pusher in production.
- Remove the network dependency from your test suite.
- You want to extend the Pusher protocol or have some special requirement. If this applies to you, chances are you are out of luck as Pusher is unlikely to implement something to suit your special use case, and rightly so. With Slanger you are free to modify and extend its behavior anyway that suits your purpose.

# Why did you write Slanger

I wanted to write a non-trivial evented app. I also want to write a book on evented programming in Ruby as I feel there is scant good information available on the topic and this project is handy to show publishers.

Pusher is an awesome service, very reasonably priced, and run by an awesome crew. Give them a spin on your next project.

# Author

- Stevie Graham

# Core Team

- Stevie Graham
- Mark Burns

# Contributors

- Stevie Graham
- Mark Burns
- Florian Gilcher
- Claudio Poli

# Maintainer
- joblee


&copy; 2020 a joblee joint.
