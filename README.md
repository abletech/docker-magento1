## Installing Magento (Instructions tested on Magento v1.8.1, and 1.9.3 -- latest)

To get a Magento 1 store running locally:

1. start docker: `docker-compose up`

If you are testing on a version other than Magento 1.9, you will need to update the docker file to point at the image you want. You can find the possible images [here](https://hub.docker.com/r/alexcheng/magento/tags)

For example:
`
  web:
    image: alexcheng/magento:1.7.0.2
`

2. You should be able to visit your Magento store at 'local.magento'. You need to update your host file to redirect localhost to local.magento
   to open the file: `sudo vim /etc/hosts`
   add: `127.0.0.1 local.magento`
   [Need help with Vim?](https://github.com/abletech/wiki/blob/8ef2a180153ad25bf3f900db85d91ae28546159c/technology_tips/beginners_guide_to_vi.md)

4. `docker-compose exec web install-magento`

5. Visit your Magento store:
  * Admin pages: http://local.magento/index.php/admin
  * Shop pages: http://local.magento/index.php/

  Credentials can be found in the env file

## Installing a Product in your store

1. Login to the admin side of your store.
2. Navigate to Catalog > Manage Products and click on add a product in the right corner.
3. Fill in the Product details. Make sure you set 'Status' to 'enabled' and 'Visibility' to 'Catalog, Search'
4. Navigate to Inventory in the left menu. Make sure the 'Quantity' is greater than zero and 'Stock Availability' is 'In Stock'
5. Navigate to Categories in the left menu. Make sure the product has a category.
6. Save your product.
7. Next, click on System > Index Management, and reindex the data. If you run into a 'try again later error' while doing this, you may need to delete the files in the var/locks folder.
8. When you search for your product in the store, you should now be able to find it and checkout

## Installing the AddressFinder Plugin Manually

1. In the Magento admin pages, click System > Cache Storage Management and click the 'Flush Magento Cache' button
2. Click System > Configuration. AddressFinder should be listed in the left menu under 'Services'
3. Click on AddressFinder in the left menu and set the 'enabled' dropdown to 'Yes'. Add any extra configuration here. If you run into issues with the page displaying a 404 not found, logging out and back in should solve it.
4. Save the config. AddressFinder should now work correctly in your store.


## Installing the AddressFinder Plugin with Composer

The docker-compose file is setup to install the plugin manually, but you will need to update to point at your addressfinder-magento-1 plugin.
If you need to install via Composer for some reason, comment out the following lines in the docker-compose file:
    ```
    - /Users/katenorquay/addressfinder/addressfinder-magento-1/app/code/community/AddressFinder:/var/www/html/app/code/community/AddressFinder:ro
    - /Users/katenorquay/addressfinder/addressfinder-magento-1/app/design/frontend/base/default/layout/addressfinder.xml:/var/www/html/app/design/frontend/base/default/layout/addressfinder.xml:ro
    - /Users/katenorquay/addressfinder/addressfinder-magento-1/app/design/frontend/base/default/template/addressfinder:/var/www/html/app/design/frontend/base/default/template/addressfinder:ro
    - /Users/katenorquay/addressfinder/addressfinder-magento-1/app/etc/modules/AddressFinder_AddressFinder.xml:/var/www/html/app/etc/modules/AddressFinder_AddressFinder.xml:ro
    - /Users/katenorquay/addressfinder/addressfinder-magento-1/js/addressfinder:/var/www/html/js/addressfinder:ro
    ```

1. `docker-compose exec web bash`
2. Inside your docker container, install composer:
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```
3. Install git inside your container:
`apt-get update`
`apt-get install git`
4. You will need to enable symlinks in your Magento store. An easy way is to install the n98-magerun plugin
`curl -O https://files.magerun.net/n98-magerun.phar`
Then enable the symlinks:
`n98-magerun.phar dev:symlinks 0`
5. install addressfinder.
`php composer.phar require addressfinder/module-magento1`
6. Install the magento-composer-installer. This creates symlinks between the vendor files created by composer, and your magento install. For this to work, you must tell composer where you have installed Magento.
`php composer.phar config extra.magento-root-dir "/var/www/html"` (Replace 'var/www/html' with your Magento insallation directory if yours is different)
`php composer.phar require magento-hackathon/magento-composer-installer:*`
7. Redeploy `php composer.phar run-script post-install-cmd -vvv -- --redeploy`
8. Follow the instructions for installing the plugin manually.