---
title: 'Upgrade from 3.0 to 3.1'
intro: 'A guide for upgrading from 3.0 to 3.1'
template: page
blueprint: page
id: 769f1c97-3fb4-4303-a60b-4096c06b7870
---
### High impact changes
- [Update Scripts](#update-scripts)
- [Collection and Nav trees](#collection-and-nav-trees)
- [Opt-in REST API Resources](#optin-rest-api-resources)

### Low impact changes
- [REST API Filtering drafts](#rest-api-filtering-drafts)
- [Entry author permissions](#entry-author-permissions)
- [Date fieldtype augmentation](#date-fieldtype-augmentation)
- [Static Caching interface](#static-caching-interface)
- [Broadcasting auth endpoint](#broadcasting-auth-endpoint)

---

## Update scripts

Statamic 3.1 introduced the concept of update scripts, which you can think of like database migrations. It'll let
Statamic perform adjustments for any particular upgrade path.

Some of the changes listed in this upgrade guide will be performed automatically by an update script.

In order for it to work, you'll need a section added to your `composer.json`.

**When upgrading using Composer, Statamic should add this automatically for you.**

(An update script adding its own update script, say whaaat?)

If it doesn't, add this `pre-update-cmd` under the `scripts` section of your `composer.json`:

```json
"scripts": {
    "pre-update-cmd": [
        "Statamic\\Console\\Composer\\Scripts::preUpdateCmd"
    ],
    ...
}
```




## Collection and Nav Trees
The trees for collections and navs are now stored separately from the collections and navs themselves.

**When upgrading using Composer, Statamic should update these automatically for you.**

If for whatever reason it doesnt, follow these steps:

If you've only got a single site:
- The `tree` array found in each `content/collections/[handle].yaml` should be moved into a `content/trees/collections/[handle].yaml` file.
- The `tree` array found in each `content/navigation/[handle].yaml` should be moved into a `content/trees/navigation/[handle].yaml` file.

If you've got multiple sites:
- In each `content/collections/[handle].yaml` with a `tree`, you should move each site's nested tree into a `content/trees/collections/[site]/[handle].yaml` file.
- Each `content/navigation/[site]/[handle.yaml]` should be moved to `content/trees/navigation/[site]/[handle].yaml` file.

Basically, the `trees` should be moved to their own dedicated directory.

## Opt-in REST API Resources
All API endpoints have been made opt-in to improve security. This means that everything will 404 until you opt into the ones you need.

Add the following array to `config/statamic/api.php` and set the resources you want to `true`.

```php
'resources' => [
    'collections' => false,
    'navs' => false,
    'taxonomies' => false,
    'assets' => false,
    'globals' => false,
    'forms' => false,
    'users' => false,
],
```

## REST API Filtering drafts
The API will now filter out draft entries.

This is more of a bug fix than a breaking change. However, if you were relying on seeing draft entries, you should be aware they won't be there now. If you want to see both, you can use the status filter:

```
/api/endpoint?filter[status:in]=published|draft
```

## Entry author permissions

3.1 introduces the ability to control how much you can control entries belonging to other users.
This feature only has any effect if your entry blueprint has an `author` field. If you don't already have an `author` field, no functionality changes for you.

If you _do_ have an `author` field, the permission logic will be enabled.

**When upgrading using Composer, Statamic should automatically apply the following updates for you.**

If for whatever reason it doesn't, you'll need to add the corresponding permissions in order to continue
editing, deleting, and managing the publish state of other users' entries:

- `edit other authors blog entries` (where `blog` is the collection's handle)
- `publish other authors blog entries`
- `delete other authors blog entries`

e.g. If you had an `edit blog entries`, you should add `edit other authors blog entries`.

## Date fieldtype augmentation
The `date` fieldtype now augments to `Carbon` instances.

If you use `{{ a_date_field }}` in Antlers without any modifiers, they will now be output using the `date_format` configured in `config/statamic/system.php` (e.g. `January 1st, 2020`).

Actual entry dates (i.e. the `{{ date }}` field) would have been formatted using the configured date format this way already. No change necessary.

If you were using a modifier (e.g. `format`), there will also be no change.

For other arbitrary date fields, the raw value (e.g. `2020-01-02`) would have been output. If you _want_ to continue to output that, you can explicitly use a modifier. e.g. `{{ a_date_field format="Y-m-d" }}`


## Static caching interface
A `hasCachedPage` method has been added to the `Statamic\StaticCaching\Cacher` interface.  

If you're making a custom Static Caching class, you'll need to add this method.
However, you're more than likely just extending `AbstractCacher`. In that case, no action is needed.


## Broadcasting auth endpoint

This only affects you if you use broadcasting *and* you've dedicided to use custom routing instead of using Laravel's `Broadcast::routes()`.

To tell Statamic about your custom auth endpoint, you should define it in `config/broadcasting.php`:

```php
'auth_endpoint' => '/your-endpoint'
```
