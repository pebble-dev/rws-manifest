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
  * `GITHUB_CONSUMER_KEY` / `GITHUB_CONSUMER_SECRET`: Used for GitHub auth (you can set them to dummy variables if you are not enabling GitHub auth in your setup).
  * `FACEBOOK_CONSUMER_KEY` / `FACEBOOK_CONSUMER_SECRET`: Used for Facebook auth (you can set them to dummy variables if you are not enabling Facebook auth in your setup).
  * `STRIPE_SECRET_KEY` / `STRIPE_PUBLIC_KEY` / `STRIPE_WEBHOOK_KEY` / `STRIPE_MONTHLY_PLAN` / `STRIPE_ANNUAL_PLAN`: Keys for Stripe; set to dummy values if you don't have a Stripe development environment.
  * `IBM_API_ROOT`: API path for weather; set to dummy value if you don't have an IBM weather key.
  * `DOMAIN_ROOT`: The root domain for RWS services.  For instance, if your boot server lives at `boot.rebble-dev.emarhavil.com`, then `DOMAIN_ROOT` should be set to `rebble-dev.emarhavil.com`.  You should have wildcard DNS set up to resolve all subdomains of `DOMAIN_ROOT` to point to your RWS Docker environment.
  * If using Algolia, `ALGOLIA_APP_ID` / `ALGOLIA_ADMIN_API_KEY` / `ALGOLIA_INDEX`; otherwise, do not add to `.env`.
* Run `docker-compose up`.  RWS should be accessible (with different services exposed selected by `Host:` headers) on port 8086.
* Set up database components:
  * Run `docker-compose exec auth flask db upgrade` to set up the database for the auth service.
  * Create OAuth keys for Rebble services to talk within themselves: `docker-compose exec auth flask create_oauth_client 'Rebble' --redirect_uri http://boot.YOUR-DOMAIN_ROOT-HERE/auth/complete --scope pebble_token --scope pebble --scope profile`
  * Append the keys to the `.env` file: set `REBBLE_CONSUMER_KEY` to the generated consumer key, and set `REBBLE_CONSUMER_SECRET` to the generated consumer secret value.
  * Run `docker-compose exec db psql -U postgres -c 'CREATE DATABASE appstore;'` to create the database for the appstore.
  * Run `docker-compose exec appstore-api flask db upgrade` to set up the database structure for the appstore.
  * Optionally, import an appstore database dump: `wget https://github.com/pebble-dev/rebble-appstore-api/releases/download/appstore-dump/import-appstore.sql && docker cp import-appstore.sql rws-compose_db_1:/ && docker-compose exec db psql -U postgres appstore -f /import-appstore.sql`
    * CAUTION: Importing this database dump will destroy any appstore data that you may already have in your local appstore database.
  * Run `docker-compose exec db psql -U postgres -c 'CREATE DATABASE timeline_sync;'` to create the database for timeline.
  * Run `docker-compose exec timeline-sync flask db upgrade` to set up the database structure for timeline.
* Optionally, punch RWS through to its own IP on port 80, if you have a spare IP address for this:
  * `# ip addr add $RWS_IP/24 dev br0`
  * `# iptables -A INPUT -d $RWS_IP -j DROP`
  * `# iptables -I INPUT -d $RWS_IP -p icmp -j ACCEPT`
  * `# iptables -I INPUT -d $RWS_IP -p tcp --dport 80 -j ACCEPT`
  * `# iptables -t nat -A PREROUTING -d $RWS_IP -p tcp --dport 80 -j REDIRECT --to-port 8086`
  * Set up wildcard DNS (for instance, `*.rebble-dev.emarhavil.com. A $RWS_IP`).
