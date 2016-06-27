## OpenShift Psiturk Cartridge

This is a fork of the [OpenShift Advanced Python Cartridge](https://github.com/gsterjov/openshift-advanced-python-cartridge) with some minimal configuration changes that make it amenable for hosting a [psiturk](https://github.com/NYUCCL/psiTurk) experiment. 

The advantages of using this cartridge over the setup described on psiturk's [readthedocs](http://psiturk.readthedocs.io/en/latest/openshift.html) are that:

1. You get an Nginx proxy server in front of the gunicorn server, which is faster for serving your static content than is the standalone psiturk gunicorn server.
2. The psiturk server starts up and runs automatically without you having to log in and start/stop it

The main disadvantage is that you can't install it using the openshift web gui -- you'll need to install the `rhc` commandline tools to install it. Instructions on doing that [here](https://developers.openshift.com/managing-your-applications/client-tools.html).

It doesn't actually run the psiturk custom gunicorn server, it uses its own gunicorn server. So your `Server Parameter` settings in your `config.txt` won't be used. However, I changed the default error log location of the cartridge's error log to be `server.log` in your repo-dir, which is the default setup in config.txt, so you don't have to go hunting for your logs. (There are other logs in ~/advanced-python/logs -- that's where the error log would have been otherwise.)

### Installation

To install this cartridge use the cartridge reflector when creating an app

	rhc create-app myapp http://cartreflect-claytondev.rhcloud.com/reflect?github=deargle/openshift-psiturk-cartridge

### Usage

You can still log in via `ssh` to the OpenShift server and start the `psiturk` shell to administrate the creation of HITs or whatever. Or you can run the `psiturk` shell from your own computer to do the administration, it doesn't make a difference as long as your AMT credentials are in place. Know though that if you start the psiturk server via `server on` that that gunicorn server won't be accessible on OpenShift because it doesn't bind to the correct port. Solution? Just don't use the `psiturk` gunicorn server! There's already one running that will server your requests. 

By default, `psiturk` is installed for you from github @master, and `psiturk-setup-example` is run for you so that you have demo working code in your repo-dir. If you want to use a different version of `psiturk`, just install it like you normally would with `pip`.

### Configuration

All you need to do is make sure to have an `app.py` file in the root of your project dir. An example is included in the repo dir that is initialized when you create an app with this cartridge. `app.py` has to import the psiturk experiment, like this:

    from psiturk.experiment import app as application

The built-in gunicorn server will know what to do from there.

### Using an Openshift-hosted DB

You can (and should) use something besides the default `sqlite` db to store your data. Consider using an Openshift-hosted relational DB. You can choose between `mysql` and `postresql`. Install them like this after you have installed the openshift-psiturk-cartridge (I've only tested that the `mysql` cartridge works):

    rhc cartridge add mysql-5.5 -a <YourAppName>

or

    rhc cartridge add postgresql-9.2 -a <YourAppName>

The database will be hosted on an `rhc` privately-accessible IP address. You can connect your local db management tools such as MySQL Workbench by setting up port forwarding. The easiest way to set up the port forwarding is to use the convenient `rhc port-forward -a <MyApp>` command. Or, you can use `phpMyAdmin` by installing another addon cartridge:

    rhc cartridge add phpmyadmin-4 -a <YourAppName>

~~**Note:** If you want to use the `openshift`-hosted DB, until my PR gets merged into the main psiturk branch, you'll need to install my PR:~~ Merged 6/27/2016


### Environment Variables

<code>OPENSHIFT_PYTHON_WORKERS</code> - The number of workers to spawn for packages like gunicorn.
Default: <code>number of CPUs * 2 + 1</code>

### Gotcha's

Recently (as of 6/27/2016), `pip install` started throwing errors on rhc. Appending `--no-cache-dir` to your command is a workaround for now.
