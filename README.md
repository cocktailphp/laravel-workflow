# Laravel workflow

Use the Symfony Workflow component in Laravel

### Installation

    composer require brexis/laravel-workflow

#### Laravel 5

Add a ServiceProvider to your providers array in `config/app.php`:

```php
<?php

'providers' => [
    ...
    Brexis\LaravelWorkflow\WorkflowServiceProvider::class,

]
```

Add the `Workflow` facade to your facades array:

```php
<?php
    ...
    'Workflow' => Brexis\LaravelWorkflow\Facades\WorkflowFacade::class,
```

Publish the config file

```
    php artisan vendor:publish --provider="Brexis\LaravelWorkflow\WorkflowServiceProvider"
```

### Configuration

Configure your workflow in `config/workflow.php`

```php
<?php

return [
    'straight'   => [
        'type'          => 'workflow', // or 'state_machine'
        'marking_store' => [
            'type'      => 'multiple_state',
            'arguments' => ['currentPlace']
        ],
        'supports'      => ['App\BlogPost'],
        'places'        => ['draft', 'review', 'rejected', 'published'],
        'transitions'   => [
            'to_review' => [
                'from' => 'draft',
                'to'   => 'review'
            ],
            'publish' => [
                'from' => 'review',
                'to'   => 'published'
            ],
            'reject' => [
                'from' => 'review',
                'to'   => 'rejected'
            ]
        ],
    ]
];
```

Use the `WorkflowTrait` inside supported classes

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Brexis\LaravelWorkflow\Traits\WorkflowTrait;

class BlogPost extends Model
{
  use WorkflowTrait;

}
```
### Usage

```php
<?php

$post = BlogPost::find(1);
$workflow = Workflow::get($post);
// if more than one workflow is defined for the BlogPost class
$workflow = Workflow::get($post, $workflowName);

$workflow->can($post, 'publish'); // False
$workflow->can($post, 'to_review'); // True
$transitions = $workflow->getEnabledTransitions($post);

// Apply a transition
$workflow->apply($post, 'to_review');
$post->save(); // Don't forget to persist the state

// Using the WorkflowTrait
$post->workflow_can('publish'); // True
$post->workflow_can('to_review'); // False

// Get the post transitions
foreach ($post->workflow_transitions() as $transition) {
    echo $transition->getName();
}

// Apply a transition
$post->workflow_apply('publish');
$post->save();
```

### Dump Workflows
Symfony workflow uses GraphvizDumper to create the workflow image. You may need to install the `dot` command of [Graphviz](http://www.graphviz.org/)

    php artisan workflow:dump workflow_name

You can change the image format with the `--format` option. By default the format is png.

    php artisan workflow:dump workflow_name --format=jpg
