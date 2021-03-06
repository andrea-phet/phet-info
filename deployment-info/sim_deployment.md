 **Simulation Deployment Guidelines**
=====================================

Variables to replace in the instructions below:

```
$SIM = the name of your sim's repo
$VERSION = the identifier of your sim, eg "1.0.0-rc.2"
$USERNAME = your username on spot and figaro
$HOME = your home directory
```

**Build process configuration**

Before building or deploying a simulation, familiarize yourself with configuration options for PhET's build process.

Your default build configuration is specified in `$HOME/.phet/build-local.json`. Describing or identifying the entries in `build-local.json` is beyond the scope of this document; ask a PhET developer for help in setting up this file. At a minimum you will need `devUsername` and `buildServerAuthorizationCode`.

Run `grunt --help` for a list of build tasks and their options. Values specified on the `grunt` command line typically override values specified in `build-local.json`.

(Optional) Create an ssh key if you'd like to avoid entering your password for dev-related build tasks:

- create an rsa key in ~/.ssh (run "ssh-keygen -t rsa" to generate a key if you don't already have one).
- add an entry for spot in ~/.ssh/config like so (you may need to create this file):

```
Host spot
   HostName spot.colorado.edu
   User [identikey]
   Port 22
   IdentityFile ~/.ssh/id_rsa
```
- On spot, you'll need to add your public key (found in ~/.ssh/id_rsa.pub) to a file ~/.ssh/authorized_keys

**Steps to publish a 'dev' (development) version**

Dev versions are deployed to spot.colorado.edu at http://www.colorado.edu/physics/phet/dev/html/.

To deploy to dev version:

1. Are your PhET repos up to date? Run pull-all.sh in chipper/bin if not.
2. Update the version identifier in package.json. The identifier should contain "dev", e.g. "1.0.0-dev.2".
3. Commit & push.
4. Run the build process: `grunt`
5. Open the generated HTML file locally and interact with it to check that all is well.
6. If this is the first time you've deployed anything for this sim, deploy with `grunt deploy-dev --mkdir` (requires 2 password inputs if ssh key is not set up). Otherwise just use `grunt deploy-dev`

**Steps to publish a 'rc' (release candidate) version**

RC versions are deployed to spot.colorado.edu at http://www.colorado.edu/physics/phet/dev/html/.

If this is the first release candidate on a release branch:

1. Create a release branch and switch to it, e.g.: `git checkout -b 1.0`. Release branches are named using major and minor version numbers, eg "1.0".
2. Update the version identifier in package.json. The first rc version should have suffix "rc.1", eg "1.0.0-rc.1".
3. Commit & push.
4. Run the build process: `grunt`
5. Deploy to the server: `grunt deploy-rc` to deploy the rc using the build server.

If this is not the first release candidate on a release branch:

1. Check out the release branch, e.g.: `git checkout 1.0`
2. Check out the correct shas for dependencies: `grunt checkout-shas`
3. If you've branched (for the purposes of patching) any of the dependency repositories since the last rc version was published, you'll need to explicitly checkout those branches. For example, if you branched vegas for the 1.1 release of graphing-lines, do `cd ../vegas ; git checkout graphing-lines-1.1`.
4. Update the version identifier in package.json. The identifier should contain "rc", e.g. "1.0.0-rc.2". Commit & push.
5. Run the build process: `grunt`
6. Deploy to the server: `grunt deploy-rc` to deploy the rc version using the build-server.
7. (optional) Check out master for dependencies: `grunt checkout-master`

**Steps to publish a public version**

Public versions are deployed to figaro.colorado.edu at http://phet.colorado.edu.
By default, publishing a public version also deploys a dev version.

To publish a public version:

1. Create a screenshot that is 2048x1344.  Generally this is done by taking a screenshot on an iPad 3 or higher (since
it has the retina display) with multiple tabs open, then crop the chrome off of it to 2048x1342, then pad the top with 2
pixels of the sim's background color (to 2048x1344) and save as /assets/$SIM-screenshot.png.  Eventually, quick and easy
creation of this screen shot will be supported through the screenshot feature from the PhET menu, but as of this writing
this support doesn't exist.
2. A public version is generally based on some rc version, which generally comes from an rc branch.  Make sure you're
working from that branch, and then update the version identifier to the form "major.minor.maintenance", removing any
"-rc.x" portion if present. For example, if the published version is based on "1.1.0-rc.4", then the published version
identifier will be "1.1.0".  Make sure this is committed and pushed.
3. Check out the correct SHAs using `grunt checkout-shas`.
4. Run `grunt` to build a local version and sanity test it.
5. Run `grunt deploy-production --locales=*` (or just `grunt deploy-production` for ph-scale and ph-scale-basics, see https://github.com/phetsims/phet-info/issues/10).  This will instruct the build server to build the English version of the
simulation as well as all currently deployed translated versions and deploy them on the web site.  This will also deploy
the locally built version to the dev server.  (Note: If you're sure that the version of chipper that is being used is
from Nov 10 2015 or later, the `--locales=*` flag can be omitted.)
6. Wait a few minutes for the build server to do its thing, and then test: http://phet.colorado.edu/sims/html/$SIM/latest/$SIM_en.html
7. If this is a new sim, both the simulation and project will need to marked "visible" in the website admin interface.
Usually the person in charge of uploading all of the meta information will be responsible for doing this. After that,
make sure the sim page appears correctly on the website. Talk to @jonathanolson or @jbphet if it hasn't appeared after marking the
sim and project visible (browse logged in to bypass the varnish cache).
8. If this is the initial publication, generate and check in (on the master branch) the auto-generated readme file for a
published sim.  The README.md file can be created using ```grunt published-README```.
9. If this is *not* a new simulation, verify that any previously existing translations are still available and that
their version numbers are correct.
10. The 3rd party contributions page must be updated.  Directions for how to do this are in reportThirdParty.js.
Brace yourself-- this will take >30 minutes since you will need to build all of the simulations in order to access
their 3rd party encumbrances.  After running this, the updated third-party-licenses.md file should be checked in to GitHub,
which will make it publicly available to people who click on "3rd party contributions" from the sim.  If this process
is too cumbersome, in the future we can discuss ways to just generate a report for the new sim and integrate it into the existing report.


**PhET-iO Deployment Guidelines**

This process will deploy a PhET simulation that is instrumented (even partially) for PhET-iO.
It places a copy on spot's dev site and a copy on figaro (not figaro-dev)

1. Update the version number in SIMULATION/package.json and commit
2. Run `grunt build-phet-io`
3. Run `grunt deploy-dev` to deploy the simulation to spot
4. Check that the `protected` directory is password protected (by .htaccess) by visiting your version of this link: `http://www.colorado.edu/physics/phet/dev/html/SIMULATION/VERSION/phet-io/protected/`
5. Copy the simulation from spot to figaro:

    1. ssh into spot
    2. scp -r /htdocs/physics/phet/dev/html/SIMULATION/VERSION USERNAME(at)figaro.colorado.edu:/data/web/htdocs/phet-io/0.1/sims/SIMULATION/VERSION

6. The .htaccess that are deployed are only suitable for spot, you must copy a working .htaccess file from concentration/1.3.0-dev.3 on figaro to the new directory:

    1. cp /data/web/htdocs/phet-io/0.1/sims/concentration/1.3.0-dev.3/phet-io/protected/.htaccess /data/web/htdocs/phet-io/0.1/sims/SIMULATION/VERSION/phet-io/protected/.htaccess
