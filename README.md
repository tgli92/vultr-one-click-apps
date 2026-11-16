# Vultr One-Click Apps: What's Preinstalled, Why Deployed Instances Never Update, and How to Size the Server

A Vultr One-Click App is a preconfigured image that puts a working application on Ubuntu Server LTS. Magento and Nextcloud both go from **Deploy Now** to a server you can log into in about a minute, and Cloudron's Marketplace page advertises **60 seconds**. You pay for the instance you pick; the Marketplace app pages don't list a separate charge for the image itself.

Three things decide whether that minute saves you an afternoon or costs you one: what the image actually installs, the fact that Vultr stops updating the application the moment it boots, and whether the instance size you chose can run the software at all.

> **Quick pick.** For a single app, take the smallest Cloud Compute instance that meets the app's published minimum (Magento states 1 GB RAM). If you want one box running several apps with certificates, DNS and backups handled for you, deploy Cloudron. If you want applications reachable through a browser with GPU acceleration, deploy WebApp.
> 👉 [Check the current Cloud Compute plan list and prices](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## What a One-Click App actually gives you

Vultr has shipped preconfigured images since January 2015, when the first four were WordPress, LEMP, Webmin and Minecraft, launched from an "Application" option on the deploy screen. The current version of that idea lives in the Vultr Marketplace, with a dedicated page for each app.

The packaging hasn't changed much: an Ubuntu Server LTS image with nginx, MySQL and the application already installed and configured far enough that you can log in and start working. What you get afterwards is root on an ordinary VPS. There is no layer between you and the server, no control panel wrapping the app itself, and no automatic scaling. That's the trade: you skip the LEMP stack build and the database initialization, then you own everything after it.

Vultr's own docs only publish a hard minimum for one app in this set. Magento requires an instance with at least 1 GB of RAM. For Nextcloud the binding constraint is disk rather than CPU, because the bundled backup script duplicates your entire installation.

| App | Runs on | What to size for | Preinstalled extras worth knowing | Deploy |
| --- | --- | --- | --- | --- |
| Magento (Community Edition) | Ubuntu Server LTS | 1 GB RAM minimum, per Vultr's docs | MySQL, PHPMyAdmin, XHProf, Cockpit on port 9080 | [Start a Magento instance](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| Nextcloud | Ubuntu Server LTS | Disk space: the backup script copies database, code and uploads into `/root` | Backup script, Nginx reset script, MySQL root password in `/root/.my.cnf` | [Start a Nextcloud instance](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| Cloudron | Ubuntu Server LTS | One instance carries every app you stack on it | Database provisioning, DNS, certificates, central users, scheduled backups, app auto-updates | [Open the Cloudron app page](https://www.vultr.com/marketplace/apps/cloudron/cloudron?ref=9738262-9J) |
| WebApp | Ubuntu Server LTS, headless | A GPU plan if you need 3D acceleration | Certbot, Cockpit on 9080, SFTP access on port 22 | [Open the WebApp app page](https://www.vultr.com/marketplace/apps/webapp?ref=9738262-9J) |

Plan names, sizes and the price for your region appear on the Cloud Compute page and again at checkout. Vultr's lineup spans shared-CPU Cloud Compute, High Frequency, Optimized Cloud Compute for CPU-, memory- and storage-heavy workloads, and GPU plans, and the entry point is a small instance measured in single-digit dollars per month. Pick by the app's requirement first, then by how much headroom you want for traffic.

## Inside the image: Magento and Nextcloud in detail

Vultr's app documentation is more specific than most, and the specifics tell you what kind of afternoon you're in for.

**Magento.** Community Edition on Ubuntu Server LTS, installed at `/var/www/html`. MySQL is running locally, and the root password sits in `/root/.my.cnf` so you don't have to hunt for it. Three browser tools come preinstalled: PHPMyAdmin at `/mysqladmin/`, XHProf profiling at `/xhprof/xhprof_html/`, and Cockpit at port 9080. Credentials for all three are printed on the Server Information page in the console. Two helper scripts save you a support ticket: `fix-vhost.sh` resets the Nginx configuration to default when a bad edit takes the site down, and `version.sh` prints the Magento and Ubuntu versions your support contact will ask for.

**Nextcloud.** The app lands in `/var/www/html` as well, and the one step people skip is the one that breaks first: Nextcloud keeps a `trusted_domains` array in `/var/www/html/config/config.php`, and a domain that isn't in it gets rejected even after DNS resolves correctly. Add your hostname there before you complain about a blank page. Certbot handles Let's Encrypt, a backup script writes database, source and uploaded files into `/root`, and there's a script that restores the default Nginx config if you corrupt it.

That backup script is worth planning around. It duplicates your data rather than archiving incrementally, so a Nextcloud instance holding 40 GB of files needs room for another 40 GB in `/root` before you run it.

## The paragraph in the docs that changes how you use these

Both the Magento and Nextcloud guides carry the same notice, and it's the part almost nobody quotes:

> "One-Click apps are updated regularly without notice. When launching a One-Click app, you'll receive our latest version. We do not update deployed instances, and you are responsible for keeping the instance up-to-date. If you design an infrastructure based on One-Click apps and need to ensure the same app version in the future, take a snapshot of the initial deployment and create new instances from the snapshot."

Read that twice. The image you deploy is current on the day you deploy it. After that, Vultr maintains the image catalogue, not your server. WordPress core, Nextcloud point releases, Magento security patches, PHP minor versions: all yours to handle.

For a personal file server, that's a monthly `apt` session and a Nextcloud updater run. For anything with plugin compatibility to protect or an audit trail to satisfy, the recommended workflow is deploy, configure, snapshot immediately, then clone from the snapshot when you need an identical second instance. Version pinning on Vultr One-Click Apps is a snapshot discipline, not a setting.

## Two things that go wrong on first login

Neither is a bug. Both are defaults you're expected to change.

**The certificate warning.** Until you point a domain at the server and install a certificate, everything is served on the raw IP with a self-signed certificate, and the browser will warn you. Vultr publishes a guide to bypassing the warning for self-signed certs, which is fine for setup and wrong for production. Register the domain, let DNS propagate (usually fast, up to 48 hours in bad cases), then run certbot.

**Magento's base URL redirect loop.** Magento installs with the IP address as its base URL. The first admin login redirects your browser back to that IP, so you get the certificate warning again even after you've set up HTTPS on the domain. Fix it in the admin under **Stores > Configuration > Web > Base URLs**: change both **Base URL** and **Base URLs (Secure)** to your domain, save, then clear the cache when Magento prompts you. After that the admin stays on the domain and the certificate reads as valid.

## Cloudron or WebApp: a panel versus a single app

These two Marketplace apps get filed together as "one-click" and behave nothing alike.

Cloudron is a multi-app platform. One instance runs WordPress, Rocket.Chat, Nextcloud, GitLab, OpenVPN and others side by side, and Cloudron handles the parts people usually get wrong: it provisions databases, configures DNS, issues and renews certificates, centralizes user accounts, takes scheduled backups, and applies upstream updates to the apps automatically. Setup starts in a browser at the server IP, where you enter a primary domain such as `example.com`. The admin interface lands at `my.example.com`, and each app gets its own subdomain (`blog.example.com`, `git.example.com`). You can attach more domains later from the Domains view.

The automatic app updates are the real differentiator. Cloudron takes on exactly the maintenance burden that Vultr's One-Click notice hands back to you.

WebApp is the opposite approach: a headless application host. You install anything from Ubuntu's APT repositories, or bring your own install script if the package doesn't exist. The application is reached through a browser at `http://use.your.ip`, and files go in over SFTP on port 22 (FileZilla works). GPU plans add 3D acceleration for gaming, CAD and scientific simulation workloads. Certbot ships installed, so a Let's Encrypt certificate is one command:

`certbot --nginx --redirect -d www.example.com -d example.com -m admin@example.com --agree-tos --no-eff-email`

Cockpit runs on port 9080 here too, and if you use Vultr's firewall you have to open 9080 explicitly or the panel simply won't load. Don't need it? `systemctl disable --now cockpit.socket` turns it off. There's also `/opt/vultr/cockpit-certificate.sh` to put the same certificate on Cockpit, and a defined revocation flow: back up `/etc/nginx`, run `certbot delete`, then `/opt/vultr/fix-vhost.sh`.

Rule of thumb: one app, pick the single-app image. Three or more apps on one box with a team behind it, pick Cloudron. Browser-delivered desktop software that needs a GPU, pick WebApp.

## Which one for your situation

**Personal file sync and sharing.** Nextcloud. Plan disk generously because of the backup script, add the domain to `trusted_domains`, run certbot, schedule the backup.

**A small store.** Magento on at least a 1 GB instance. A 1 GB server satisfies the requirement and leaves little room for a catalogue, so treat it as the floor rather than the target. Fix the base URL before you import products.

**A team running several tools.** Cloudron. The subdomain structure and centralized users save more time than the deployment speed does.

**Graphics or engineering software in a browser.** WebApp on a GPU plan, with the firewall opened on 9080 if you want Cockpit.

**A blog or a plain LEMP stack.** The single-app WordPress or LEMP image, smallest instance that fits, done.

## Before you click Deploy

Two checks. First, confirm the instance meets the app's stated requirement, because Magento at 1 GB and Magento at 1 GB with a live catalogue are different things. Second, decide now whether you need a fixed version; if you do, take the snapshot immediately after configuration, before you install a single plugin, and build future servers from that snapshot rather than from the app catalogue.

The link that brought you here advertises a $300 code credit for new accounts. Read the terms on that page before you plan around it, since eligibility, expiry and which products the credit applies to are set by the promotion, not by the app you deploy.

👉 [See the current sign-up credit terms and Cloud Compute pricing](https://bit.ly/VuLtr)
