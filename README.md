# edX Ecommerce Installation / Configuration Documentation

## Ecommerce Setup
Ecommerce is preinstalled as part of the standard native Open edX build. However, you still have to configure internal oauth and configure the products that you will be offering to learners. You should also at least be aware that we made minor customizations to the source code for Rover. These include:
  - changes to edx-platform fork to modify Ecommerce marketing text to change "Verified Certificate" stuff to a "Rover Paid Seat".
  - changes to edx-theme to modify some of the Ecommerce-related UI elements.
  - there are NO CHANGES to the ecommerce repo

Ecommerce configuration is spread across
  - LMS Django
  - Ecommerce django
  - configuration files located in /edx/etc
  - configuration files located in /edx/app/ecommerce/ecommerce/ecommerce/settings/production.py

If you're behind a load balancer then there are a few extra configuration requirements.

### Payment providers (Stripe, PayPal)
Most of the ecommerce payment service configuration is stored in /edx/etc/ecommerce.yml.
* Stripe test credit card numbers: https://stripe.com/docs/testing#cards

### LMS configuration
1. Bug fix: if after attempting to edit/save Ecommerce Course Admin records you
   see errors in the LMS log with
     File "/edx/app/edxapp/venvs/edxapp/local/lib/python2.7/site-packages/django/db/models/query.py", line 386, in get()
     DoesNotExist: UserProfile matching query does not exist.
   Then no user Profile record exists for the ecommerce_worker user. to resolve
   you should open the user in Django admin and populate a few data fields in the
   "Profile" section, and then save the record.

2. add OIDC/AUTH -- /admin/oauth2/client/1/
![Django admin oauth](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/django-admin-oauth.png "Django admin oauth")

3. verify /admin/commerce/commerceconfiguration/

4. edit the following in lms.env.json
```json
"FEATURES": {
  "ENABLE_OAUTH2_PROVIDER": true
}
"BASE_COOKIE_DOMAIN": "[SUBDOMAIN].roverbyopenstax.org",
"SESSION_COOKIE_DOMAIN": "[SUBDOMAIN].roverbyopenstax.org",
"SITE_NAME": "[SUBDOMAIN].roverbyopenstax.org",
"CMS_BASE": "[SUBDOMAIN].roverbyopenstax.org",
"ECOMMERCE_PUBLIC_URL_ROOT": "http://[SUBDOMAIN].roverbyopenstax.org:18130",
"JWT_ISSUER": "http://[SUBDOMAIN].roverbyopenstax.org/oauth2",
  "LMS_BASE": "http://[SUBDOMAIN].roverbyopenstax.org",
  "LMS_ROOT_URL": "http://[SUBDOMAIN].roverbyopenstax.org",
  "OAUTH_OIDC_ISSUER": "http://[SUBDOMAIN].roverbyopenstax.org/oauth2",

"CROSS_DOMAIN_CSRF_COOKIE_DOMAIN": "[SUBDOMAIN].roverbyopenstax.org",

    "DEFAULT_JWT_ISSUER": {
        "AUDIENCE": "SET-ME-PLEASE",
        "ISSUER": "http://[SUBDOMAIN].roverbyopenstax.org/oauth2",
        "SECRET_KEY": "SET-ME-PLEASE"
    },

 "ECOMMERCE_API_URL": "http://[SUBDOMAIN].roverbyopenstax.org:18130/api/v2",

    "JWT_AUTH": {
        "JWT_AUDIENCE": "SET-ME-PLEASE",
        "JWT_ISSUER": "http://[SUBDOMAIN].roverbyopenstax.org/oauth2",
        "JWT_ISSUERS": [
            {   
                "AUDIENCE": "SET-ME-PLEASE",
                "ISSUER": "http://[SUBDOMAIN].roverbyopenstax.org/oauth2",
                "SECRET_KEY": "SET-ME-PLEASE"
            }
        ],
        "JWT_SECRET_KEY": "SET-ME-PLEASE"
    },
```

5. Verify LMS oauth Trusted Clients at /admin/edx_oauth2_provider/trustedclient/
![LMS oauth trusted clients](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/lms-trusted-clients.png "LMS oauth trusted clients")

6. Setup LMS oauth clients for ecommerce and discovery at /admin/oauth2/client/
![LMS oauth clients](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/lms-oauth-clients.png "LMS oauth clients")

7. Verify permissions of LMS ecommerce_worker user at /admin/auth/user/
 - Make this user "Staff" and "Superuser"
 - Change email address to support@roverbyopenstax.org
![LMS ecommerce_worker user](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/lms-ecom_worker-user.png  "LMS ecommerce_worker user")

### Ecommerce configuration
1. edit /edx/etc/ecommerce.yml from this repo and copy to server
    replace {CLIENT}.roverbyopenstax.org with your fully-qualified domain name
    do not changes references to localhost

2. edit and execute rover/scripts/edx.ecommerce-install.sh
3. edit /edx/app/ecommerce/ecommerce/ecommerce/settings/production.py
    if necessary change mysql host name to internal IP address of remote mysql server instance
    #on or around row 86

```python
    DB_OVERRIDES = dict(
        PASSWORD=environ.get('DB_MIGRATION_PASS', DATABASES['default']['PASSWORD']),
        ENGINE=environ.get('DB_MIGRATION_ENGINE', DATABASES['default']['ENGINE']),
        USER=environ.get('DB_MIGRATION_USER', DATABASES['default']['USER']),
        NAME=environ.get('DB_MIGRATION_NAME', DATABASES['default']['NAME']),
        HOST=environ.get('DB_MIGRATION_HOST', '172.31.6.65'),
        PORT=environ.get('DB_MIGRATION_PORT', DATABASES['default']['PORT']),
    )
```    

4. Run db migrations
![Ecommerce db migrations](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-db-migrations.png "Ecommerce db migrations")

5. Create a Ecommerce superuser account
```python
sudo su ecommerce -s /bin/bash
cd ~/ecommerce
source ../ecommerce_env
python ~/ecommerce/manage.py createsuperuser
```

6. Create a Ecommerce Partner record at :18130/admin/partner/partner/1/change/
![Ecommerce Partner](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-django-partner.png "Ecommerce Partner")

7. Verify Ecommerce configuration at :18130/admin/core/siteconfiguration/1/change/
![Ecommerce configuration](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-config-1.png "Ecommerce configuration")
![Ecommerce configuration](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-config-2.png "Ecommerce configuration")
![Ecommerce configuration](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-config-3.png "Ecommerce configuration")

8. Verify Ecommerce Site Configuration at :18130/admin/sites/site/1/change/
![Ecommerce configuration](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-site-config.png "Ecommerce configuration")



### Ecommerce UI elements to verify
If you configured everything successfully then you'll see marketing blurbs on the LMS student dashboard course cards as well as in the Course "About" page for each course which you created a product in Oscar.

![LMS Student Dashboard](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/lms-dashboard.png "LMS Student dashboard")
![LMS Course Info](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/lms-course-info.png "LMS Course info")
![Ecommerce checkout basket](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-checkout-basket.png "Ecommerce checkout basket")
![Ecommerce checkout receipt](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/checkout-receipt.png "Ecommerce checkout receipt")


### Oscar Ecommerce product configuration & management pages
![Course admin page](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-course-admin.png "Course admin page")
![Ecommerce Oscar products](https://github.com/lpm0073/edx-ecommerce/blob/master/doc/ecommerce-oscar-products.png "Ecommerce Oscar products")


### Ecommerce theming
https://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/ecommerce/theming.html


### more info:
- https://openedx.atlassian.net/wiki/spaces/OpenOPS/pages/110330276/How+to+Install+and+Start+the+E-Commerce+Service+in+Native+Installations
- https://blog.lawrencemcdaniel.com/open-edx-ecommerce/


## Discovery Service
Documentation: https://edx-discovery.readthedocs.io/en/latest/introduction.html
- DO NOT FOLLOW oauth setup instructions: https://edx-discovery.readthedocs.io/en/latest/advanced.html
  these instructions are not correct.

  oauth setup is identical to that of Ecommerce. all of the settings are stored in /edx/etc/discovery.yml
  the oauth provider configuration is done in /admin/oauth2/client/

- edit /edx/etc/discovery.yml
    EDX_DRF_EXTENSIONS:
        OAUTH2_USER_INFO_URL: http://127.0.0.1:8000/oauth2/user_info

    ELASTICSEARCH_URL: http://127.0.0.1:9200/

    JWT_AUTH:
    JWT_ISSUERS:
    -   AUDIENCE: SET-ME-PLEASE
        ISSUER: http://127.0.0.1:8000/oauth2
        SECRET_KEY: SET-ME-PLEASE

        SOCIAL_AUTH_EDX_OIDC_ID_TOKEN_DECRYPTION_KEY: discovery-secret
    SOCIAL_AUTH_EDX_OIDC_ISSUER: http://127.0.0.1:8000/oauth2
    SOCIAL_AUTH_EDX_OIDC_KEY: discovery-key
    SOCIAL_AUTH_EDX_OIDC_LOGOUT_URL: http://127.0.0.1:8000/logout
    SOCIAL_AUTH_EDX_OIDC_PUBLIC_URL_ROOT: http://127.0.0.1:8000/oauth2
    SOCIAL_AUTH_EDX_OIDC_SECRET: discovery-secret
    SOCIAL_AUTH_EDX_OIDC_URL_ROOT: http://127.0.0.1:8000/oauth2
    SOCIAL_AUTH_REDIRECT_IS_HTTPS: false


- setup ssl https on nginx
- setup trusted client in LMS django admin oauth2 / clients (screen shot)
- setup a partner in https://[SUBDOMAIN].roverbyopenstax.org:18381/admin/core/partner/
- ensure that the following urls work:
  - https://[SUBDOMAIN].roverbyopenstax.org:18381/api/v1/courses/
  - https://[SUBDOMAIN].roverbyopenstax.org:18381/admin/
  - https://[SUBDOMAIN].roverbyopenstax.org:18381/api-docs/
