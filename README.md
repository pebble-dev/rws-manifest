# RWS boot manifest for `repo`

Quick "getting started" guide:

* Install `repo` on your system.  (On OS X, `brew install repo` will do the right thing.)
* `repo init -u git@github.com:pebble-dev/rws-manifest.git`.  Or, if you prefer HTTPS, `repo init -u https://github.com/pebble-dev/rws-manifest.git`
* `repo sync`

To get RWS running:

* Install `docker` and `docker-compose` on your system.
  * If you're running on Mac OS X, you may need to set up `docker-machine`.  Doing so is outside of the scope of this documentation.
* `cd` into `TOT/dev/rws-compose` (where `TOT` refers to the top of your tree -- that is to say, the directory that you did the `repo init` in).
* Create a `.env` file with global environment variables for your RWS install.  You will need, at least, the following:
  * `SECRET_KEY`: A securely random string.
  * `GOOGLE_CONSUMER_KEY` / `GOOGLE_CONSUMER_SECRET`: Used for Google auth (you can set them to dummy variables if you are not enabling Google auth in your setup).
  * `TWITTER_CONSUMER_KEY` / `TWITTER_CONSUMER_SECRET`: Used for Twitter auth (you can set them to dummy variables if you are not enabling Twitter auth in your setup).
  * `DOMAIN_ROOT`: The root domain for RWS services.  For instance, if your boot server lives at `boot.rebble-dev.emarhavil.com`, then `DOMAIN_ROOT` should be set to `rebble-dev.emarhavil.com`.  You should have wildcard DNS set up to resolve all subdomains of `DOMAIN_ROOT` to point to your RWS Docker environment.
* Run `docker-compose up`.  RWS should be accessible (with different services exposed selected by `Host:` headers) on port 80.
