![Kirby Modules](https://user-images.githubusercontent.com/7975568/93752618-37d29000-fbff-11ea-8276-abd679ef92ae.png)

This plugin makes it super easy to create modular websites with Kirby.

## Features

- Modules are bundled in `site/modules` and registered as regular blueprints and templates.
- Every module is available to create in the `modules` section without editing any other file.
- Modules can not be accessed directly and will automatically redirect to the parent page with an anchor.
- The container page is automatically created and hidden in the panel.
- You can preview draft modules on their parent pages via the panel preview button.

![Preview](https://user-images.githubusercontent.com/7975568/94016693-bb7eaf00-fdae-11ea-8114-f0862391ff91.gif)

## What's a Module?

A module is a regular page, differentiated from other pages by being inside a modules container.
This approach makes it possible to use pages as modules without sacrificing regular subpages.

```
📄 Page
  📄 Subpage A
  📄 Subpage B
  🗂 Modules
    📄 Module A
    📄 Module B
```

## Instructions

Add a `modules` section to any page blueprint and a modules container will be automatically created.
 
Similar to [blocks](https://getkirby.com/docs/reference/panel/fields/blocks), you can create module blueprints in `site/blueprints/modules/` and module templates in `site/snippets/modules/`. E.g. `site/blueprints/modules/text.yml` and `site/snippets/modules/text.php`.

It's also possible to use a separate `site/modules/` folder. In this case, you create your module blueprint in `site/modules/text/text.yml` and the module template in `site/modules/text/text.php`.
 
In the parent page template you can then use `<?= $page->modules() ?>` to render the modules.

### Parent Page

#### `site/blueprints/pages/default.yml`

```yml
title: Default Page
sections:
  modules: true
```

#### `site/templates/default.php`

```php
<?= $page->modules() ?>
```

### Example Module

#### `site/blueprints/modules/text.yml`

```yml
title: Text Module
fields:
  textarea: true
```

#### `site/snippets/modules/text.php`

```php
<div class="<?= $module->moduleId() ?>" id="<?= $module->uid() ?>">
  <h1><?= $module->title() ?></h1>
  <?= $module->textarea()->kt() ?>
</div>
```

You can access the module page object with `$module` and the parent page object with `$page`.
The `$module->moduleId()` method returns the module ID, e.g. `module_text` or `module_gallery`.

### Skipping the delete confirmation dialog

Adding a page model for the parent page(s) and overwriting the `hasChildren` method skips the confirmation dialog you see when deleting a page with children. You can adjust the code depending on what you want to happen when there are modules or just a modules container:

```php
<?php

use Kirby\Cms\Page;

class DefaultPage extends Page {
  public function hasChildren(): bool {
    $children = $this->children()->filterBy('intendedTemplate', '!=', 'modules');
    $children = $children->merge($this->grandChildren());
    $children = $children->merge($this->children()->drafts());
    return $children->count() > 0;
  }
}
```

Thanks to [@lukaskleinschmidt](https://github.com/lukaskleinschmidt) for helping me with this.

## Options

### Default Module Blueprint

By default, the `text` module will be the first/default option in the "Add page" modal.
You can overwrite it in your `site/config/config.php`:

```php
return [
  'medienbaecker.modules.default' => 'gallery'
];
```

### Exclude Module Blueprints

By default, all modules will be available in the "Add page" modal.
You can exclude certain modules in your `site/config/config.php`:

```php
return [
  'medienbaecker.modules.exclude' => [
    'hero',
    'anotherForbiddenModule'
  ]
];
```

### Autopublish Modules

You can turn on automatic publishing for modules in your `site/config/config.php`:

```php
return [
  'medienbaecker.modules.autopublish' => true
];
```

### Custom Module Model

This plugin creates a `ModulePage` model, overwriting certain methods.
You can extend this model with your own model:

```php
// site/config/config.php

return [
  'medienbaecker.modules.model' => 'CustomModulePage'
];
```

```php
// site/models/module.php

class CustomModulePage extends ModulePage {
  // methods...
}
```

## Installation

Download this repository to `/site/plugins/kirby-modules`.

Alternatively, you can install it with composer: `composer require medienbaecker/kirby-modules`

## License

This project is licensed under the terms of the MIT license.
