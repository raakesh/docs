# Creating Packages

## Creating Packages

### Introduction

Creating packages is very simple for Masonite. You can get a package created and on PyPi is less than 5 minutes. With Masonite packages you'll easily be able to integrate and scaffold all Masonite projects with ease. Masonite comes with several helper functions in order to create packages which can add configuration files, routes, controllers, views, commands and more.

### Getting Started

As a developer, you will be responsible for both making packages and consuming packages. In this documentation we'll talk about both. We'll start by talking about how to make a package and then talk about how to use that package or other third party packages.

Masonite, being a Python framework, can obviously utilize all Python packages that aren’t designed for a specific framework. For example, Masonite can obviously use a library like requests but can’t use Django Rest Framework.

Similarly to how Django Rest Framework was built for Django, you can also build packages specific to Masonite. Although you can just as simply build packages for both, as long as you add some sort of Service Provider to your package that can integrate your library into Masonite.

#### About Packages

There are several key functions that Masonite uses in order to create applications. These include primarily: routes, controllers, views, and craft commands. Creating a package is simple. Conveniently Masonite comes with several helper functions in order to create all of these.

You can easily create a command like `craft mypackage:install` and can scaffold out and install controllers, routes, etc into a Masonite project.

You do not have to use this functionality and instead have the developer copy and paste things that they need to from your documentation but having a great setup process is a great way to promote developer happiness which is what Masonite is all about.

#### Creating a Package

Like other parts of Masonite, in order to make a package, we can use a craft command. The `craft package` command will scaffold out a simple PyPi package and is fully able to be uploaded directly to PyPi.

This should be done in a separate folder outside of your project.

Let's create our package:

```text
$ craft package testpackage
```

This will create a file structure like:

```text
testpackage/
    __init__.py
    integration.py
MANIFEST.in
setup.py
```

#### **Creating a Config Package**

Lets create a simple package that will add or append a config file from our package and into the project.

First lets create a config file inside `testpackage/snippets/configs/services.py`. We should now have a project structure like:

```text
testpackage/
    __init__.py
    integration.py
    snippets/
        configs/
            services.py
MANIFEST.in
setup.py
```

Great! Inside the `services.py` lets put a configuration setting. This configuration file will be directly added into a Masonite project so you can put doctrings or flagpole comments directly in here:

```text
TESTPACKAGE_PAYMENTS = {
    'stripe': {
        'some key': 'some value',
        'another key': 'another value'
    }
}
```

Perfect! Now we'll just need to tell PyPi to include this file when we upload it to PyPi. We can do this in our `MANIFEST.in` file.

```text
include testpackage/snippets/configs/*
```

#### **Creating an Install Command**

It's great \(and convenient\) to add craft commands to a project so developers can use your package more efficiently. You can head over to [Creating Commands](../the-craft-command/creating-commands.md) to learn how to create a command. It only involves a normal command class and a Service Provider.

Head over to that documentation page and create an `InstallCommand` and an `InstallProvider`. This step should take less than a few minutes. Once those are created we can continue to the adding package helpers below.

#### **Adding Migration Directories**

Masonite packages allow you to add new migrations to a project. For example, this could be used to add a new `package_subscriptions` table if you are building a package that works for subscribing users to Stripe.

Inside the Service Provider you plan to use for your package we can register our directory:

```python
from masonite.provider import ServiceProvider

package_directory = os.path.dirname(os.path.realpath(__file__))

class ApiProvider(ServiceProvider):

    def register(self):
        self.app.bind(
            'TestPackageMigrationDirectory',
            os.path.join(package_directory, '../migrations')
        )
```

Masonite will find any keys in the container that end with `MigrationDirectory` and will add it to the list of migrations being ran whenever `craft migrate` and `craft migrate:*` commands are ran.

The `package_directory` variable contains the absolute path to the current file so the migration directory being added should also be an absolute path to the migration directory as demonstrated here. Notice the `../migrations` syntax. This is going back one directory and into a migration directory there.

#### **Package Helpers**

Almost done. Now we just need to put our `masonite.package` helper functions in our install command. The location we put in our `create_or_append_config()` function should be an absolute path location to our package. To help with this, Masonite has put a variable called `package_directory` inside the `integration.py` file. Our handle method inside our install command should look something like:

```python
import os
from cleo import Command
from masonite.packages import create_or_append_config


package_directory = os.path.dirname(os.path.realpath(__file__))

class InstallCommand(Command):
    """
    Installs needed configuration files into a Masonite project

    testpackage:install
    """

    def handle(self):
        create_or_append_config(
            os.path.join(
                package_directory,
                '../testpackage/snippets/configs/services.py'
            )
        )
```

{% hint style="warning" %}
**Make sure this command is added to your Service Provider and the developer using your package adds it to the** `PROVIDERS` **list as per the** [**Creating Commands**](../the-craft-command/creating-commands.md) **documentation.**
{% endhint %}

This will append the configuration file that has the same name as our package configuration file. In this case the configuration file we are creating or appending to is `config/services.py` because our packages configuration file is `services.py`. If we want to append to another configuration file we can simply change the name of our package configuration file.

#### **Working With Our Package**

We can either test our package locally or upload our package to PyPi.

To test our package locally, if you use virtual environments, just go to your Masonite project and activate your virtual environment. Navigate to the folder where you created your package and run:

```text
$ pip install .
```

If you want to be able to make changes without having to constantly reinstall your package then run

```text
$ pip install --editable .
```

This will install your new package into your virtual environment. Go back to your project root so we can run our `craft testpackage:install` command. If we run that we should have a new configuration file under `config/services.py`.

#### **Uploading to PyPi**

If you have never set up a package before then you'll need to [check how to make a `.pypirc` file](http://peterdowns.com/posts/first-time-with-pypi.html). This file will hold our PyPi credentials.

To upload to PyPi we just have to pick a great name for our package in the `setup.py` file. Now that you have a super awesome name, we'll just need to run:

```text
$ python setup.py sdist upload
```

which should upload our package with our credentials in our `.pypirc` file. Make sure you click the link above and see how to make once.

If `python` doesn’t default to Python 3 or if PyPi throws errors than you may need to run:

```text
$ python3 setup.py sdist upload
```

#### **Consuming a package.**

Now that your package is on PyPi we can just run:

```text
$ pip install super_awesome_package
```

Then add your Service Provider to the `PROVIDERS` list:

```python
from testpackage.providers.InstallProvider import InstallProvider
PROVIDERS = [
    ...
    # New Provider
    InstallProvider(),
]
```

and then run:

```text
$ craft testpackage:install
```

Remember our Service Provider added the command automatically to craft.

Again, not all packages will need to be installed or even need commands. Only packages that need to scaffold the project or something similar need one. This should be a judgment call on the package author instead of a requirement.

You will know if a package needs to be installed by reading the packages install documentation that is written by the package authors.

## Publishing

Masonite has the concept of publishing packages. This allows you to manage the integration with your package and Masonite in a more seamless way. Publishing allows you to add things like routes, views, migrations and commands easily into any Masonite app and it is all handled through your service provider

The goal is to have a developer run:

```bash
$ craft publish YourProvider
```

This should be the name of your provider class.

and have all your assets moved into the new Masonite application.

### Publishing Files

You can create or append any files you need to in a developers masonite application. This can be used for any files to include commands, routes, config files etc.

For example let's say you have a directory in your package like:

```text
validation/
  providers/
    ValidationProvider.py
  commands/
    RuleCommand.py
setup.py
```

Inside our service provider we can do this:

```python
import os

class ValidationProvider(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self):
        command_path = os.path.join(os.path.dirname(__file__), '../commands')

        self.publishes({
            os.path.join(command_path, 'RuleCommand.py'): 'app/commands/RuleCommand.py'
        })
```

Notice our command path is 1 directory back inside the `commands` directory. We then combine the directory with the `RuleCommand.py` file and tell Masonite to put it inside the `app/commands/RuleCommand.py` module inside the users directory.

The user of your package will now have a new command in their application!

### Publishing Migrations

You can take any migrations in your package and send them to the Masonite applications migration directory. This is useful if you want to have some developers edit your custom migrations before they migrate them.

For example let's say you have a directory in your package like:

```text
validation/
  providers/
    ValidationProvider.py
  migrations/
    user_migration.py
    team_migration.py
setup.py
```

The migrations like `user_migration.py` should be full migration files.

Then you can have a service provider like this:

```python
import os

def boot(self):
    migration_path = os.path.join(os.path.dirname(__file__), '../migrations')

    self.publishes_migrations([
        os.path.join(migration_path, 'user_migration.py'),
        os.path.join(migration_path, 'team_migration.py'),
    ])
```

This will create a new migration in the users directory.

### Publishing Tags

You can also add tags to each of these migrations as well. For example if you have 2 sets of migrations you can do this instead:

```python
import os

def boot(self):
    migration_path = os.path.join(os.path.dirname(__file__), '../migrations')
    command_path = os.path.join(os.path.dirname(__file__), '../commands')

    self.publishes({
        os.path.join(command_path, 'RuleCommand.py'): 'app/commands/RuleCommand.py'
    }, tag="commands")

    self.publishes_migrations([
        os.path.join(migration_path, 'user_migration.py'),
        os.path.join(migration_path, 'team_migration.py'),
    ], tag="migrations")
```

Now a user can only either publish migrations or commands by adding a `--tag` option

```bash
$ craft publish ValidationProvider --tag migrations
```

This will ignore the commands publishing and only publish the migrations

