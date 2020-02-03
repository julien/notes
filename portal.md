# Liferay portal installation

This is a guide to setup [liferay-portal](https://github.com/liferay/liferay-portal) on your development machine.

For the moment this document is "macOS" (and MacOS X) specific but I'll add notes
for linux and Windows later on.

**IMPORTANT:**

Read this document first and if you think you need help don't hesitate to ask.


## Basic Requirements

This guide assumes that:

- [Git](https://git-scm.com) is installed and configured on your machine.

- You have a [GitHub](https://github.com) account and are part of the [liferay](https://github.com/liferay) organization.

- You have the correct permissions to install software on your machine.

- You have a few GB's (at least 3-4GB) of storage avaialbe on your HDD.

## Getting the source code

1. Login to your GitHub account.

2. Fork the [liferay-portal](https://github.com/liferay/liferay-portal) repository to your account.

3. Create a `portal` directory anywhere of your choosing:

    ```shell
    mkdir -p portal
    ```

4. Clone the `liferay-portal` repository in the `portal` directory create above

    ```shell
    # This might seem obvious but just in case it isn't,
    # replace YOUR_GITHUB_USERNAME with your GitHub user name.

    git clone git@github.com:YOUR_GITHUB_USERNAME/liferay-portal.git

    # NOTE: This might take a long time so make sure your machine doesn't
    #       enter sleep mode and that your internet connection is working correctly
    ```

   **NOTE:**

   Don't worry if you've already cloned the `liferay-portal` git repository somewhere else,
   you can always do this before compiling the source code, or even later on.

## Installing the required software

You'll also need to have the following tools installed on your machine

- [Apache Ant](https://ant.apache.org/)

- [Java JDK8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

- [MySQL 5.7](https://dev.mysql.com/downloads/mysql/5.7.html)

- [Homebrew](https://brew.sh)

- [nodejs](https://nodejs.org/en/download/)


**IMPORTANT**:

If your know how to install these tools yourself, feel free to skip this section.


We'll go throught them one by one:

### Homebrew

On macOS (and MacOS X) we'll be using Homebrew to install Apache Ant and MySQL.

If the `brew` command is not availalbe on your system, please install Hombrew with:

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Wait for everything to termiate correctly.

### Java JDK8

Open your browser and visit [this](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) URL.

You'll need to accept the license agreement (by clicking in a radio button
somewhere next to the downloads list)  before you can
download anything.

Once that's done, you can download the Java SE Development package for your
operating system.

There are usually 2 versions (the two latest released) on the page, choose the
"newest" one and execute the installer once download is complete.

Follow the installation process and make sure everything completes succesfully.

Open a terminal and make sure you can execute the `java` command

```shell
java -version

# This should print something similar to:
# java version "1.8.0_152"
# Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
# Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)
```

#### Using the Open JDK

In theory, you may be able to use the [Open JDK](https://jdk.java.net/) 11 with Liferay, installed with Homebrew (`brew cask install java`). In practice I found that the version that it currently installs produces an error message ("Please use Java 1.8.") when you attempt to build. If you already have this JDK installed on your system you may have to remove it (`brew cask uninstall java`) and install the Oracle version as described above instead.

### nodejs

Open your browser and visit [this](https://nodejs.org/en/download/) URL.

Download the installer for your operating system, execute the installer and make sure everything completes succesfully.

Open a terminal and make sure you can execute the `node` and `npm` commands

```shell
node -v

# This should print something similar to:
# v10.15.0

npm -v

# This should print something similar to:
# 6.4.1
```

You'll also want to make sure you can install **global** dependencies.

```shell
npm i -g gh gh-jira gradle-launcher
```

If the above fails because of wrong permissions or if you'd like to use another directory for your global modules, please read [this](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

### Apache Ant and MySQL 5.7

Open a terminal and issue the following command:

```shell
brew install ant mysql@5.7
```

**IMPORTANT:**

If the above fails, ask for help.

If you have a pre-existing MySQL installation — or a related one, such as MariaDB — on your system, you may need to uninstall it before proceeding. For example, I had a MariaDB installation that I had to remove (`brew uninstall mariadb`) as well as blowing away the contents of `/usr/local/var/mysql` because the files there were preventing the server from starting with errors in the `/usr/local/var/mysql/$(hostname).err` log files such as "InnoDB: Unsupported redo log format. The redo log was created with MariaDB".

#### Verifying the installation

Wait for everything to complete correctly and make sure you have access to both the `ant` and `mysql` commands.

```shell
ant -v

# This should print something similar to:
# Apache Ant(TM) version 1.10.5 compiled on July 10 2018
# Trying the default build file: build.xml
# Buildfile: build.xml does not exist!
# Build failed
```

```shell
mysql --version

# This should print something similar to:
# mysql  Ver 14.14 Distrib 5.7.22, for osx10.11 (x86_64) using  EditLine wrapper
```

Make sure that you can start the MySQL server correctly

```shell
brew services start mysql@5.7
```

You might also want to secure your `mysql` installation by running the following command:


```shell
mysql_secure_installation

# Answer the questions:
# Remove test database and users
# Disallow remote login
# Reload privileges
```

You'll also need to create a database for `liferay-portal`, we typically name it `lportal_master`.

**IMPORTANT:**

If you choose another name for your database the following instructions will need to be updated
to match your chosen database name. Be warned.

```shell
mysql -uroot -p
# Enter your password

# Create the DB
CREATE DATABASE `lportal_master` DEFAULT CHARACTER SET utf8;

# Exit mysql
exit
```

## Configuration

You'll also need a couple of environment variables defined:

```shell
export ANT_OPTS="-Xmx4096m"
export JAVA_HOME=$(/usr/libexec/java_home)
export JAVA_OPTS="-Xmx4096m" # You could probably re-use $ANT_OPTS here
```

We'll also create a `portal-ext.properties` file with our portal's preferences:

In the `portal` directory you created above, you need to create a `bundles` sub-directory:

```shell
cd portal
mkdir -p bundles
```

Inside this `bundles` directory, create a file named `portal-ext.properties` with the following content:

```properties
## Database Configuration
jdbc.default.driverClassName=com.mysql.jdbc.Driver

## Make sure this matches the database name you created
## and your MySQL credentials!
jdbc.default.url=jdbc:mysql://localhost/lportal_master?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
jdbc.default.username=YOUR_MYSQL_USERNAME
jdbc.default.password=YOUR_MYSQL_PASSWORD

## Development Configuration
com.liferay.portal.servlet.filters.cache.CacheFilter=false
com.liferay.portal.servlet.filters.alloy.CSSFilter=false
com.liferay.portal.servlet.filters.etag.ETagFilter=false
com.liferay.portal.servlet.filters.header.HeaderFilter=false
com.liferay.portal.servlet.filters.themepreview.ThemePreviewFilter=true
combo.check.timestamp=true
javascript.fast.load=false
javascript.log.enabled=false
layout.template.cache.enabled=false
minifier.enabled=false
minifier.inline.content.cache.size=0
module.framework.properties.lpkg.index.validator.enabled=false
module.framework.properties.osgi.console=localhost:11311
```

## Compiling

Make sure you have been able to clone the `liferay-portal` repository before you start with this step.

Your `portal` directory should now look like this:

```
- portal
 |
 |- bundles
 |  |
 |  |- portal-ext.properties
 |
 |- liferay-portal
 |  |
 |  |- lots of directories and files ...

```

Navigate to the `liferay-portal` directory and run the following command:

```shell
ant all

# This should take about 30-40 minutes the first time you do it on your machine
# It should also end with a "BUILD SUCCESSFUL" message, but if it doesn't
# ask @julien for some help
```

Once that's done, you can launch `tomcat` with the following command:


```shell
# Assuming your are in the 'portal/liferay-portal' directory


# You'll need to replace
# tomcat-VERSION with the tomcat version that comes with liferay-portal.
# Tip: Use the tab key assuming your shell has tab completion.

../bundles/tomcat-VERSION/bin/catalina.sh run
```

This should start `tomcat` on port `8080`.

If everything is OK and once the server is started a browser window will be opened pointing at `http://localhost:8080`.

Since this is the first time you're executing `liferay-portal` you'll have to answer a few questions in a wizard.

We typically use "test" as the user name and password in development mode but you can choose anything you want.

Once that's done, you'll need to stop the `tomcat` server process and start it again.

**IMPORTANT:**

If you don't want a new browser tab/window to be opened each time you start `tomcat`, you can add

```properties
browser.launcher.url=
```

To the `portal-ext.properties` file your created.

### Optional: Speed up compilation by cloning [liferay-binaries-cache-2020](https://github.com/liferay/liferay-binaries-cache-2020)

This repo contains pre-built binaries of the third-party dependencies of the portal. If you clone it as a peer of your `bundles` directory, it will speed up compilation:

```sh
cd ..
git clone https://github.com/liferay/liferay-binaries-cache-2020
```

After this, your directory structure should look like:

```shell

- portal
 |
 |- bundles
 |  |
 |  |- portal-ext.properties
 |  |- ...
 |
 |- liferay-binaries-cache-2020
 |  |
 |  |- ...
 |
 |- liferay-portal
 |  |
 |  |- ...
```

## Last steps

Change defaut tomcat settings by editing the `CATALINA_OPTS` environment
variable. To do so, edit the `setenv.sh` file.

```shell
# You'll need to replace
# tomcat-VERSION with the tomcat version that comes with liferay-portal.
# Tip: Use the tab key assuming your shell has tab completion.

$EDITOR bundles/tomcat-VERSION/bin/setenv.sh
```

These are some recommended settings for development

```shell
CATALINA_OPTS="$CATALINA_OPTS -Dfile.encoding=UTF8 -Djava.net.preferIPv4Stack=true -Dorg.apache.catalina.loader.WebappClassLoader.ENABLE_CLEAR_REFERENCES=false -Duser.timezone=GMT -Xms4096m -Xmx4096m -XX:MaxNewSize=2048m -XX:MaxMetaspaceSize=768m -XX:MetaspaceSize=768m -XX:NewSize=2560m -XX:SurvivorRatio=7"
```

Add an `upstream` remote to your local `liferay-portal` repository that points to `liferay` 's repo on GitHub.

```shell
git remote add upstream git@github.com:liferay/liferay-portal.git
```

Configure the `gh` utlity:

Run `gh us` to create the following configuration file named `.gh.json` in your home directory. If that doesn't work you can create the file manually with these contents:

```json
{
  "version": "1.11.4",
  "api": {
    "host": "api.github.com",
    "protocol": "https",
    "version": "3.0.0",
    "pathPrefix": null
  },
  "default_branch": "master",
  "default_remote": "origin",
  "default_pr_forwarder": "",
  "default_pr_reviewer": "",
  "github_token": "",
  "github_user": "",
  "hooks": {
    "issue": {
      "close": {
        "before": [],
        "after": []
      },
      "new": {
        "before": [],
        "after": [
          "gh is --browser {{options.browser}} --user {{options.user}} --repo {{options.repo}} --number {{options.number}}"
        ]
      },
      "open": {
        "before": [],
        "after": []
      }
    },
    "pull-request": {
      "close": {
        "before": [],
        "after": []
      },
      "fetch": {
        "before": [],
        "after": [
          "gh pr {{options.number}} --user {{options.user}} --repo {{options.repo}} --comment 'Just started reviewing :)'"
        ]
      },
      "fwd": {
        "before": [],
        "after": [
          "gh pr {{options.submittedPullNumber}} --user {{options.fwd}} --comment '/cc @{{options.submittedUser}}'",
          "gh pr {{options.number}} --user {{options.user}} --repo {{options.repo}} --comment 'Pull request forwarded to {{forwardedLink}}.{{#if options.changes}} [See changes here.]({{compareLink}}){{/if}}'",
          "gh pr {{options.number}} --close"
        ]
      },
      "merge": {
        "before": [],
        "after": [
          "gh pr {{options.number}} --user {{options.user}} --repo {{options.repo}} --comment 'Thank you, pull request merged!{{#if options.changes}} [See changes here.]({{compareLink}}){{/if}}'"
        ]
      },
      "open": {
        "before": [],
        "after": []
      },
      "submit": {
        "before": [],
        "after": [
          "{{#if options.number}}gh pr {{options.number}} --user {{options.user}} --repo {{options.repo}} --comment 'Pull request submitted to {{submittedLink}}.{{#if options.changes}} [See changes here.]({{compareLink}}){{/if}}'{{/if}}",
          "gh pr --browser {{options.browser}} --user {{options.submit}} --repo {{options.repo}} --number {{options.submittedPull}}",
          "{{#if options.number}}gh pr --user {{options.user}} --repo {{options.repo}} {{options.number}} --close{{/if}}"
        ]
      }
    },
    "repo": {
      "delete": {
        "before": [],
        "after": []
      },
      "fork": {
        "before": [],
        "after": []
      },
      "new": {
        "before": [],
        "after": [
          "gh re --browser {{options.browser}} --user {{options.user}} --repo {{options.new}}"
        ]
      }
    },
    "gists": {
      "delete": {
        "before": [],
        "after": []
      },
      "fork": {
        "before": [],
        "after": [
          "gh gi --browser {{options.browser}} --id {{options.id}}"
        ]
      },
      "new": {
        "before": [],
        "after": [
          "gh gi --browser {{options.browser}} --id {{options.id}}"
        ]
      }
    }
  },
  "ignored_plugins": [],
  "pull_branch_name_prefix": "pr-",
  "plugins": {
    "jira": {
      "host": "",
      "user": "",
      "password": "",
      "base": "rest/api/2"
    }
  },
  "replace": {},
  "signature": " <br><br>:octocat: *Sent from [GH](http://nodegh.io).*",
  "plugins_path": "FULL_PATH_TO_YOUR_NODE_INSTALLATION/lib/node_modules"
}
```

**IMPORTANT:**

Be sure to replace `FULL_PATH_TO_YOUR_NODE_INSTALLATION` with the absolute path to your nodejs installation directory
or you'll get a warning about `gh` not being able to read it's plugin directory.


In your terminal try logging into your GitHub account with `gh` by running the following command:

```shell
gh us
```

You'll be prompted to enter your GitHub user name and password and if everything works fine your credentials will be stored.

You can also configure the `gh-jira` plugin to be able to manage Jira related tasks from the command line.

Enter the following command in your terminal:

```shell
gh ji
```

**IMPORTANT:**

When prompted for the server, enter "issues.liferay.com"

Here's an example of how you would create a task from the command-line, :

```shell
gh jira --new \
  --project IFI \
  --title "Set up liferay-portal developer environment" \
  --message 'Going through set-up documented in https://github.com/julien/notes/blob/master/portal.md and creating PRs along the way for any issues discovered in the notes.' \
  --type Task
```

You should see output like the following:

```
Creating a new issue on project IFI
https://issues.liferay.com/browse/IFI-422
```

For more details, see [the gh-gira documentation](https://www.npmjs.com/package/gh-jira).

## Well done

That's it, you should be ready to work on liferay portal, have fun.
