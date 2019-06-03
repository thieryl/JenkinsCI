#!/bin/bash

# ----------------------------------------------------
# v. 20180308
TARGET=/var/www/html
TMPTARGET=/var/www/codedeploy
BACKUP=/var/www/html-backup
USER=apache
GROUP=apache
# ----------------------------------------------------

echo "Temporary folder: $TMPTARGET"
echo "Workspace folder: $WORKSPACE"

echo "Cleaning temporary folder"
sudo rm -rf $TMPTARGET

echo "Creating the required symbolic links"
sudo rm -rf $TMPTARGET/media
sudo ln -s $TARGET/../media $TMPTARGET/
sudo rm -rf $TMPTARGET/var
sudo ln -s $TARGET/../var $TMPTARGET/

echo "Changing permissions of the temporary folder to apache user"
cd $TMPTARGET
mkdir -p $TARGET/../env/media; chown $USER:$GROUP $TARGET/../env/media;
mkdir -p $TARGET/../env/var; chown $USER:$GROUP $TARGET/../env/var;
find $TMPTARGET -type d -exec chmod 775 {} \;
find $TMPTARGET -type f -exec chmod 664 {} \;
chown -R $USER:$GROUP $TMPTARGET

echo "MAINTENANCE STARTS"
sudo chmod a+x ./bin/magento
sudo php ./bin/magento maintenance:enable || true

echo "Files replacement"
rm -rf $BACKUP
mv $TARGET $BACKUP
mv $TMPTARGET $TARGET
cd $TARGET
rm -rf pub/media; sudo -H -u $USER bash -c "ln -s $TARGET/../env/media ./pub/media  || true"
rm -rf var; sudo -H -u $USER bash -c "ln -s $TARGET/../env/var ./var  || true"

# SELINUX security
sudo chcon -t httpd_sys_content_t $TARGET -R
sudo chcon -t httpd_sys_rw_content_t $TARGET/../env/ -R

echo "Composer auth.json"
cd $TARGET
mkdir -p /var/www/.composer
mv $TARGET/deployment/auth.json /var/www/.composer/auth.json
chown $USER:$GROUP /var/www/.composer/auth.json

echo "Dependencies"
cd $TARGET
sudo -H -u apache bash -c "composer install --no-dev"
cd update
sudo -H -u apache bash -c "composer install --no-dev"

echo "Flush cache"
cd $TARGET
sudo rm -rf generated/* var/cache var/di var/generation var/page_cache

echo "Setup upgrade"
cd $TARGET
chmod a+x ./bin/magento
sudo -H -u www-data bash -c "./bin/magento setup:upgrade"

echo "Production mode"
sudo -H -u apache bash -c "./bin/magento deploy:mode:set production -s"
sudo -H -u apache bash -c "./bin/magento setup:di:compile"
sudo rm -rf pub/static/* var/view_preprocessed
sudo -H -u apache bash -c "./bin/magento setup:static-content:deploy en_US"
sudo -H -u apache bash -c "./bin/magento cache:flush"

sudo -H -u apache bash -c "./bin/magento deploy:mode:show"

echo "MAINTENANCE STARTS"
sudo chmod a+x ./bin/magento
sudo php ./bin/magento maintenance:disable || true

echo "Healthcheck file"
cp $TARGET/deployment/configs/healthcheck.html $TARGET/
cp $TARGET/deployment/configs/healthcheck.html $TARGET/pub/

echo "Remove sensitive scripts"
rm -rf $TARGET/deployment

echo "Hurray!!!!"
