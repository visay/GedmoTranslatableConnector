# TYPO3 Flow Connector to Gedmo Translatable

by Sebastian Kurfürst, sandstorm|media. Thanks to WebEssentials for sponsoring this work.

Using Gedmo.Translatable in TYPO3 Flow proved a little harder than originally anticipated -- that's why I created
a small package which wraps up the necessary steps.

Has been tested on recent TYPO3 Flow master; should also work on Flow 2.2.


## Usage

Just include this package, and then use Gedmo Translatable as explained in their documentation (e.g. using
the @Gedmo\Translatable annotation): https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/translatable.md

Make sure to clear the code cache completely in Data/Temporary after installing this package!

Furthermore, make sure to create a *doctrine migration* which creates the ext_translations SQL table; e.g. run `./flow doctrine:migrationgenerate`

**Check out the example package at https://github.com/sandstorm/GedmoTest**.

### Model Annotations

Just annotate your model properties which shall be localized with `Gedmo\Mapping\Annotation\Translatable`.

### Translating a model (low-level)

```
	/**
	 * Doctrine's Entity Manager. Note that "ObjectManager" is the name of the related interface.
	 *
	 * @Flow\Inject
	 * @var ObjectManager
	 */
	protected $entityManager;

	public function updateAction(Event $event) {
		/* @var $repository TranslationRepository */
		$repository = $this->entityManager->getRepository('Gedmo\\Translatable\\Entity\\Translation');
		$repository->translate($event, 'name', 'de', 'Deutscher Titel');
	}
```

### Set current language

In order to set the current language for *viewing*, inject the `Gedmo\Translatable\TranslatableListener` class and set
the current language on it: `$translatableListener->setTranslatableLocale('de');`.

### Editing multiple languages at the same time

* Mix-in the Trait `Sandstorm\GedmoTranslatableConnector\TranslatableTrait` into your model, e.g. by doing:

```
/**
 * @Flow\Entity
 */
class MyModel {
  use \Sandstorm\GedmoTranslatableConnector\TranslatableTrait;
  
  // make sure some properties have Gedmo\Translatable annotations
}
```

* This trait adds a `getTranslations()` and `setTranslations()` method, allowing to get and set other translations of
  a model.
  
* Now, you can easily edit multiple languages by binding the form element to `translations.[language].[fieldname]`, e.g.
  this works like the following:

```
Name (default): <f:form.textfield property="name" /><br />
Name (de): <f:form.textfield property="translations.de.name" /><br />
Name (en): <f:form.textfield property="translations.en.name" /><br />
```

### Fetching an object in another locale

If you have loaded an object in a specific locale, but lateron need to change the object to be in another locale,
the method `reloadInLocale($locale)` (which is defined inside the trait `Sandstorm\GedmoTranslatableConnector\TranslatableTrait`)
can be called:

```
$myModel->getName(); // will return the language which was set at the time where $myModel was fetched

$myModel->reloadInLocale('de');
$myModel->getName(); // will return *german*
```

**NOTE**: This has only been tested as long as these objects are not updated.


## Inner Workings

(as a further reference -- could also be reduced if we change TYPO3 Flow a little on the relevant parts)

* Settings.yaml: Ignore Gedmo namespace from Reflection, adds the Translatable Listener as Doctrine Event listener.
  This part is pretty standard for using other Gedmo Doctrine extensions as well.
  
* Objects.yaml: Mark the `TranslatableListener` as singleton, such that you can inject it into your classes and set
  the current language. This part was straightforward as well.
  
* Package.php: Make the entities of Gedmo Translatable known to Doctrine and in Reflection. This was quite tricky to
  archive, see the inline docs in the class how this was done.


## TODO list

* check TYPO3 Flow 2.2