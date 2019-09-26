- Start Date: 2019-09-26
- RFC PR: (leave this empty)

# Summary

Ability to provide a file, command line argument, environment variable, or combination of the above to specify files ignored by the live reload process in `strapi develop`.

# Example


1. There could be a configuration file:

`.strapiignore` with the content:

```
database
```

Where in this case any folder named database would be ignored.

2. This could be accomplished in a similar manner to how Nodemon does it with [config files](https://github.com/remy/nodemon#config-files).

`strapi-config.js`:

```
"ignore": ["*.test.js", "fixtures/*"],
```

3. Or how it does it with [arguments](https://github.com/remy/nodemon#ignoring-files).

```
strapi develop --ignore lib/ --ignore tests/
```

# Motivation

There are times where teams will need to place other files in the Strapi root. Especially with `strapi-docker` where some files can be changed during heavy development time but do not need to restart the Strapi CMS.

A specific use case would be database seeding, whereby teams have setup a database seed script that configures aspects of Strapi that are stored in the database, or seeds collections of objects in the CMS.

# Detailed design

Describe the proposal in details:

A user would create a file, define some aruguments specifying files or folders that should be ignored by Strapi Develop. Strapi would join/merge these config options and add them as additional ignore paths to Chokidar.

On the [develop commands](https://github.com/strapi/strapi/blob/master/packages/strapi/lib/commands/develop.js#L97) the Chokidar watcher would merge in additional ignores:

```
const cliIgnores = <process arguments script>;
const fileIgnores = <process to read ignores from config file>;

const watcher = chokidar.watch(dir, {
  ignoreInitial: true,
  ignored: [
  	...[
	    /(^|[/\\])\../,
	    /tmp/,
	    '**/admin',
	    '**/admin/**',
	    '**/components',
	    '**/components/**',
	    '**/documentation',
	    '**/documentation/**',
	    '**/node_modules',
	    '**/node_modules/**',
	    '**/plugins.json',
	    '**/index.html',
	    '**/public',
	    '**/public/**',
	    '**/cypress',
	    '**/cypress/**',
	    '**/*.db*',
	    '**/exports/**',
  	],
  	...cliIgnores,
		...fileIgnores,
	],
});
```

# Tradeoffs

It would require additional documentation.
It would require additional file i/o and cli i/o processing to gather the additional ignore files.
It cannot be implemented outside of the Strapi core package.
I think this proposal is relatively segmented from any current proposals.
I don't think this is a breaking change.
It would require `strapi-docker` to pass through or allow configuration of the ignore files from the Docker CLI, these would then need to be passed through the [strapi.sh](https://github.com/strapi/strapi-docker/blob/master/strapi.sh) script.

# Alternatives

Alternatively the Strapi script could be passed a set of watch files to reload on, rather than a set of files to ignore. But there is already a large set of ignored files so a exclusive rather than inclusive pattern makes more sense.

# Unresolved questions

Specific integration and choice of pattern to use (CLI, file, environment etc) which would be based on the preference of the core team and ease of implementation. Given Strapi Docker uses the startup script it I would lean towards command arguments being a easier solution to implement.
