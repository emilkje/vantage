
# Vantage

[<img src="https://travis-ci.org/dthree/vantage.svg" alt="Build Status" />](http://travis-ci.org/dthree/vantage)

Same application. Brand new point of view.

    npm install vantage -g

Vantage provides a distributed, interactive command-line interface to your live Node application or web server.

####Doesn't commander.js do that?

Yes, and no. Inspired by and based on [commander.js](https://www.npmjs.com/package/commander), Vantage allows you to connect into and hop between running Node applications with an interactive prompt provided by [inquirer.js](https://www.npmjs.com/package/inquirer), giving a real-time perspective of your application you otherwise haven't had.

The CLI includes built-in help, command history and autocompletion.

```bash
$ npm install vantage -g
$ vantage 127.0.0.1:80
$ Connecting to 127.0.0.1:80 using http...
$ Connected successfully.
myapp~$ 
myapp~$ debug on -v 7
Turned on debugging with verbosity to 7.
... [live logging] ...
...
...
myapp~$ 
```

####Contents

* Getting Started
* Basic Commands
  - command
    - description
    - alias
    - action
  - delimiter
  - listen
  - prompt
  - exec
  - prompt
  - pipe
  - use
* Automation
* Firewall
* Authentication
* Extensions

##Getting Started

Setting up a basic application:

```js
// server.js
var Vantage = require('vantage');
var server = new Vantage();

server
  .command('foo')
  .description('Outputs "bar".')
  .action(function(args, cb) {
    console.log('bar');
    cb();
  });
  
server
  .delimiter('webapp~$')
  .listen(80)
  .prompt();
```

You can now run it directly. The `server.prompt()` command enables the prompt from the terminal that started the application:

```bash
$ node server.js
webapp~$ 
```

With `server.listen(80)` given above, you can remotely connect to the application from another terminal:

```bash
$ vantage 80
$ Connecting to 127.0.0.1:80 using http...
$ Connected successfully.
webapp~$ 
```
You can now execute your application's CLI commands remotely, and the `stdout` from the application will pipe to your terminal:

```bash
webapp~$
webapp~$ foo
bar
webapp~$
```

A built-in help lists all available commands:

```bash
webapp~$ help

  Commands
  
    help [command]    Provides help for a given command.
    exit [options]    Exists instance of Vantage.
    vantage [server]  Connects to another application running vantage.
    foo               Outputs "bar".

webapp~$
```

## Basic Commands

###.command(command, [description])

Adds a new command to your command line API. Returns a `Command` object, with the following chainable functions:

`.description(string)`: Used in automated help for your command.

`.alias(string)`: Aliases to execute the same command.

`.option(string, [description])`: Provides command options, as in `-f` or `--force` or `--no-cheese`.

`.action(function)`: Function to execute when command is executed.

#####Command Syntax

The syntax is similar to `commander.js` with the exception of allowing nested sub-commands for grouping large APIs into managable chunks. Examples:

```js
vantage.command('foo'); // Simple command with no arguments.
vantage.command('foo [bar]'); // Optional argument.
vantage.command('foo <bar>'); // Required argument.

// Example of nested subcommands:
vantage.command('farm animals');
vantage.command('farm tools');
vantage.command('farm feed [animal]');
vantage.command('farm sit with farmer brown and reflect on <subject>');
```
When displaying the help menu, sub-commands will be grouped separately:

```bash
webapp~$ help

  Commands:
  
     ...
  
  Command Groups:
  
    farm *            4 sub-commands.
    
webapp~$
```

Entering `farm` or `farm --help` would then drill down on the commands:

```bash
webapp~$ farm

  Commands:
  
    farm animals        Lists all animals in the farm.
    farm tools          Lists all tools in the farm.
    farm feed [animal]  Feeds a given animal.
  
  Command Groups:
  
    farm see *          1 sub-command.
    
webapp~$
```
