# Configuration Export for CiviCRM

## Status

Development of this extension is paused. If you're looking at doing something in this space, I'd be interested to discuss - hit me (Chris Burgess, @xurizaemon) up. Good places to do this are [CiviCRM's "dev" channel](https://chat.civicrm.org/civicrm/channels/dev), or anywhere you feel like.

## What did we learn?

* CiviCRM can reliably import and export entities to a config format. That works fine.
* CiviCRM has some "dependency" issues which would need to be mapped in order to do this reliably. (eg: A contribution page may only be imported / exported reliably if it can be mapped to other entities, like memberships, related organisations, price sets ...)
* Mapping these dependencies would be a time-consuming task.
* Mapping other constraints (eg some entities must have unique names - I forget which, but I noted it somewhere) would also be required.
* Handling situations where these expectations are not met might be complicated, and some of the logic to ensure these constraints are met lives in Civi's forms layer.
* CiviCRM's `hook_civicrm_managed` will currently remove the managed entities. It would be nice to have a means to "disown" or "free" the managed entities so that removing this extension doesn't blow away a ton of the site's configuration - otherwise that could be really disastrous.
* We learned that this could be a big task, for now.

## TODO

* Rename as "ConfigManagement" or something, because (duh!) Import is not Export, and we came here to Import too.
* Rename as "EntityManagement" or something, because (duh!) not everything is Config, and I realised we can export content like Contacts too. (Although the `civicrm_managed` table is gonna get *pretty weird* doing that.)
* Identify issues where entities must be unique in characteristics other than ID (eg, "Payment processor name must be unique").
* Identify issues where dumping / loading entities triggers mismatches (financial_type_id needs to be rewritten in import to connect to correct financial type).

## API additions

Get a UUID for something (this makes it a managed thing in passing). Probably want to be able to look for UUIDs without making entities managed?

    drush cvapi uuid.get entity_type=payment_processor_type entity_id=16

Export a something. Stores it to ConfigAndLog/configmanager/ENTITY_TYPE/UUID.yml

    drush cvapi configexport.export entity_type=contact entity_id=219

## Dependencies

Dependencies are exported as part of the parent object, like so:

    ---
    uuid: 557f9364-93f8-4598-b3ab-454beb60fb72
    title: Help Support CiviCRM!
    intro_text: >
      Do you love CiviCRM? Do you use CiviCRM?
      Then please support CiviCRM and
      Contribute NOW by trying out our new
      online contribution features!
    financial_type_id: 1
    payment_processor: 1
    currency: USD
    configmgr_dependencies:
      payment_processor:
        -
          uuid: 07ed40f0-4328-4eb3-b389-8301e49051d1
          domain_id: 1
          name: Test Processor
          payment_processor_type_id: 10
          url_site: http://dummy.com
          class_name: Payment_Dummy
          payment_type: 1
      financial_type:
        -
          uuid: f32219e5-4b5c-49d5-8de6-0e5791f46b90
          name: Donation
          is_active: 1

Nested dependencies are not yet handled, and dependent entities still export some details which they ought not to. We're into realms where we need to deal with specifics of CiviCRM now, eg "Two entities of type X may not share a name" and so forth.

## Installation

If you've got the code from Github, you'll also need to run `composer install` in the extension directory to install required libraries.

If distributed, this step will be removed. For now, consider it a "you must be this high to ride" constraint :)

When the extension is installed, core schema will be modified when a `civicrm_managed.uuid` column will be added.

## Disabling

**Entities managed by this module will automatically be removed when it is disabled.**

## Uninstallation

Uninstalling the module will remove the `civicrm_managed.uuid` column and `civicrm_configexport` table.

## Questions

* Modifying core schema is discouraged, and maybe we could / should do this without modifying civicrm_managed table ... but this will do for now.
