# Extension Management

Normally all extensions installed by the Administration will be stored inside `custom/plugins` or `custom/apps`. When you want to update extensions, you have to re-upload the zip file or download the extension from the store using the Extension manager in the administration.

This way of extension management brings many problems:

* It is hard to keep track of which extensions are installed and in which version
* The extensions can be modified live in the Administration without version control
* Extension updates must be downloaded manually for each extension and installed
* Extension updates in the Administration can't be done together with Shopware updates
* Composer class loader cannot be optimized because we need to dynamically look up into `custom/plugins`

## Installing extensions with Composer

To solve these problems, it is recommended to install all extensions (plugins and apps) with Composer. This way, you can manage all extensions in one place and update them along with Shopware. To get started with Composer, first, you need to authorize your local project with the Shopware Composer Registry. Below are the steps:

* Login to [account.shopware.com](https://account.shopware.com) and go to your Shop (in Merchant or Account area)
* Click on one extension
* Click the button "Install via Composer"
* Generate a token and save it

Now you can add the Shopware Composer Registry to your project:

```bash
composer config repositories.shopware-packages '{"type": "composer", "url": "https://packages.shopware.com"}'

composer config bearer.packages.shopware.com <your-token>
```

After that, you should have a newly created file `auth.json`, in your project root. This file contains your token and is used by Composer to authenticate against the Shopware Composer Registry.

{% hint style="info" %}
The `auth.json` should not be committed to the repository and should be ignored by default with the `.gitignore` file.
{% endhint %}

Now you can install extensions with Composer:

```bash
composer require store.shopware.com/{extension-name}
```

You can also find the Composer package name when you click "Install via Composer" in the Shopware Account.

## Migrating already installed extensions to Composer

If you already have extensions installed in your project, you can migrate them to Composer. First, you should install the extension with Composer:

```bash
composer require store.shopware.com/{extension-name}
```

And then delete the source code from `custom/plugins/{extension-name}` or `custom/apps/{extension-name}`.

After that, you must run the below command for Shopware to detect the installed extensions per Composer.

```bash
bin/console plugin:refresh
```

## Enabling Composer class map authoritative

When all extensions are installed with Composer, you can enable the Composer class map authoritative. This will improve the performance of the class loader and is recommended for production environments.
[The class map authoritative, disables the live class lookup when it cannot find the class in a dumped class map.](https://getcomposer.org/doc/articles/autoloader-optimization.md#optimization-level-2-a-authoritative-class-maps)

```diff
{
    "require": {
        "shopware/core": "....",
        // .....
    },
    "config": {
        "optimize-autoloader": true,
+       "classmap-authoritative": true
    }
}
```

And run the below command to re-generate the class loader.

```bash
composer dump-autoload
```
