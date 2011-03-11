================================================================================
Préparing Release and Patch
================================================================================

(Temporary until I write it up proper)

0. Run tests on a PostgreSQL DB and probably MySQL

1. Change MARKUS_VERSION

2. Update NAMED_REVISIONS

3. Make sure INSTALL is up-to-date in the repository and a most recent version
is exported on markusproject.org (i.e. /path/to/www/INSTALL)

3. Make sure [deployment instructions](wiki:InstallProdStable) match latest
requirements

3. Update Changelog

  ``svn log --verbose --stop-on-copy`` is your friend.

4. Make sure that config/environments/production.rb has reasonable settings

5. Commit version/release files changes to repo

6. If it is possible, run rake i18n:missing_keys and add them to locales (you
can keep the english key if you don't know how to translate it.) It will avoid
missing locales errors.

7. Package everything up and put it on markusproject.org

8. Update the symlink on markusproject.org in the path/to/www/download/ folder
(markus-latest-stable.tar.gz)