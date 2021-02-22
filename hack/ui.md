# CloudStack UI Development

For the modern UI, starting 4.15, refer to the following:
- https://github.com/apache/cloudstack/tree/master/ui#development
- https://github.com/apache/cloudstack/blob/master/ui/docs/development.md

**NOTE**: The CloudStack legacy UI (up to version 4.15) is a single page legacy-styled static
JavaScript app based on jQuery. This is only valid for CloudStack versions upto 4.15.
Since 4.16, the legacy UI has been deprecated and removed.

Javascript, jQuery references:
- https://www.w3schools.com/js/
- https://www.geeksforgeeks.org/javascript-tutorial/
- https://learn.jquery.com/
- https://www.w3schools.com/jquery/
- https://www.tutorialspoint.com/jquery/

All the source is in the `ui` directory with its entry point at `index.html`,
the following tree shows the UI codebase filesystem:

```bash
    ui
    ├── css           # CSS files
    ├── images        # sprites, icons, images
    ├── l10n          # translation files
    ├── lib           # jquery lib files
    ├── modules       # UI modules/components for specific features
    ├── plugins       # UI plugins
    └── scripts       # UI components
        └── *.js
        └── ui        # UI framework, widgets such as dialogs, tables etc.
        └── ui-custom # Customized feature specific UI widgets
    ├── ...
    └── index.html    # Main html file
```

Case study: CloudStack Metrics Views https://github.com/apache/cloudstack/pull/1038

## Development

During development, when CloudStack is fully built the UI assets are copied to
`client/target/classes/META-INF/webapp/`. UI development can be extremely
inefficient if developers rebuild and stop/start management server using `mvn`
everytime they make changes. Instead developers can use an iterative-styled
development approach which requires changes files in the `client/target` path
directly and perform a hard-refresh in browser to test/iterative changes.
For example:

```bash
    cd client/target/classes/META-INF/webapp
    # make changes to the UI, hard-refresh UI instead of restarting mgmt server
    # iterate your implementation
    # finally copy the changes back to `ui` directory
    cp -rv . ../../../../../ui/
    git add -p # check and stage UI changes
```

## Debugging

You can use browser specific dev-tools to debug the UI, css, js code, put break
points, hot edit/load JS code, use its console. In the js code, you can log a
statement or object by using `console.log(argsHere);` method.

References:
- https://developers.google.com/web/tools/chrome-devtools/
- https://developer.mozilla.org/son/docs/Tools

## Implementation

For a new resource, its UI can be implemented either in `scripts` (for example,
see `ui/scripts/roles.js` and include any new file(s) in index.html) or as a
separate UI plugin under `ui/plugins/` (for example, see `ui/plugins/testPlugin`).

CloudStack UI implements its own jQuery based framework and supports UI plugins
following the following structure:

```bash
    ui
    └── plugins
        └── plugins.js  # configure and enable your plugin here by the name
        └── coffee
            └── coffee.css # css rules
            └── coffee.js  # js code
            └── config.js  # plugin config file
            └── icon.png   # typically a 50x50 px size tab icon
```

You can approach UI plugin implementation by referencing the `testPlugin` or
other UI plugins. Your UI plugin's `config.js` should something like the
following:

```javascript
(function (cloudStack) {
  cloudStack.plugins.coffee.config = {
    title: 'Coffee',
    desc: 'CloudStack Coffee',
    externalLink: 'http://example.com',
    authorName: 'Your Name',
    authorEmail: 'yourname@example.com'
  };
}(cloudStack));
```

Notice that your plugin should pick a unique non-spaced name (usually lowercase)
such as `coffee` and it should attach the plugin object to the `cloudStack`
object using `cloudStack.plugins.coffee`.

Suitably create `icon.png`, `coffee.css`, and `coffee.js` files.

The plugin `js` file structure defines what the plugin exports. The CloudStack
UI framework allows following to be implemented:
- `sections` that are drop downs using which a view can be changed (for example,
  see the global settings tab)
- `listView` (a table) with `fields` (columns), `actions` (buttons) that can
  have form based input, notifications etc, and `dataProvider` that can call
  APIs to populate the table
- `detailView` is a view/page describing an item from the `listView` (table)
  that can have paths, actions (buttons to perform action on that resource item)
  and tabs that typically have a `Detail` tab that lists various attributes of
  that item; and can have `dataProvider` as well

Skeletal structure of plugin implementation describing the plugin, sections and
a list view of a section:

```javascript
(function (cloudStack) {
    cloudStack.plugins.coffee = function(plugin) {
        plugin.ui.addSection({
          // important to declare a unique `id`
          id: 'coffee',
          title: 'Coffee',
          preFilter: function(args) {
                // Place check to show/hide this plugin
                return true;
          },
          showOnNavigation: true,
          sectionSelect: {
              label: 'label.select-view',
              preFilter: function(args) {
                  // sections to show based on some conditional
                  return ['coffee'];
              }
          },
          sections: {
              coffee: {
                  // important to declare a unique `id`
                  id: 'coffee',
                  type: 'select',
                  title: 'label.coffee',
                  listView: {
                      section: 'coffee',
                      // important to declare a unique `id` for the listView
                      id: 'coffee',
// .. code redacted ..
}(cloudStack));
```

A list view can describe its fields (columns of the table), actions,
dataprovider and the detail view. For example:

```javascript
id: 'coffee',
label: 'label.coffee',
fields: {
    name: {
        label: 'label.name'
    },
    account: {
        label: 'label.account',
        truncate: true
    },
    state: {
        label: 'label.state',
        // a special field `state` can show various icons based on the state
        // using `indicator` one can define which state shows which icon
        // on: green, off: red, warning: orange, default is a grey icon
        indicator: {
            'Brewing': 'warning',
            'Brewed': 'on',
            'Removed': 'off'
        }
    }
},
actions: {
    // define listView actions here
},
dataProvider: function(args) {
    // define how to want to display the data
    // data can be set to the table using `args.response.success()`
    // in case of an error `args.response.error()` can be used
},
detailView: {
    // define detail view here
}
```

A `listView` action generally tries to add a resource, and can be defined with
a form to take inputs. For example:

```javascript
add: {
    label: 'label.add.coffee',
    preFilter: function(args) {
        return true;
    },
    messages: {
        notification: function(args) {
            return 'label.add.coffee';
        }
    },
    createForm: {
        title: 'label.add.coffee',
        desc: 'message.add.coffee',
        fields: {
            name: {
                label: 'label.name',
                validation: {
                    required: true
                }
            },
            type: {
                label: 'label.type',
                validation: {
                    required: true
                }
            },
            checkbox: {
                label: 'label.some.checkbox',
                isBoolean: true,
                isChecked: true
            }
        }
    },
    action: function(args) {
        // log args to see what you can use
        // checkbox can be checked using `args.data.checkbox == 'on' ? true: false`
        console.log(args);
        var data = {
            name: args.data.name,
            type: args.data.type
        };
        $.ajax({
            url: createURL('createCoffee'),
            data: data,
            success: function(json) {
                var item = json.createcoffeeresponse.coffee;
                args.response.success({
                    data: item
                });
            },
            error: function(json) {
                args.response.error(parseXMLHttpResponse(json));
            }
        });
    },
    notification: {
        poll: function(args) {
            args.complete();
        }
    }
}
```

A listView `dataProvider` can make an `$.ajax` call to make an API call, retrieve
the response json and set that so the table can be populated.

Reference: http://api.jquery.com/jquery.ajax

For example:

```javascript
dataProvider: function(args) {
    console.log(args); // log args to see what you can use
    $.ajax({
        url: createURL('listCoffees'),
        data: {
            page: args.page,
            pagesize: pageSize,
            keyword: args.filterBy.search.value
        },
        success: function(json) {
            var items = json.listcoffeesresponse.coffee;
            args.response.success({
                data: items
            });
        },
        error: function(json) {
            args.response.error(parseXMLHttpResponse(json));
        }
    });
}
```

A `detailView` can define actions (buttons to perform action on resource
item), paths (to navigate to other views) and tabs to display row based
key/value attributes and custom UI widgets.

Typical structure can look as follows:

```javascript
name: 'label.coffee.details',
viewAll: [{
    // define paths here, for example:
    path: 'accounts',
    label: 'label.accounts'
}],
actions: {
    // define actions here, for example:
    edit: {
      label: 'label.update',
      action: function(args) {
          console.log(args);
          // Code to make ajax call
              // Nested code to handle API response, set jobId for async API
              // UI can poll for async API result using the jobId via the queryAsyncJobResult API
              // For example:
              args.response.success({
                  _custom: {
                      jobId: jId
                  }
              });
        },
        notification: {
            // only used for polling async API result
            poll: pollAsyncJobResult
        }
    }
},
tabs: {
    details: {
        // Define the default `details` tab and others, for example:
        title: 'label.details',
        fields: [{
            id: {
                label: 'label.id'
            }
        }, {
            name: {
                label: 'label.name',
                // editable fields will show up as inputs fields for a special
                // action/button `edit` defined in the detailView
                isEditable: true
            },
            account: {
                label: 'label.account'
            },
            state: {
                label: 'label.state'
            },
            created: {
                label: 'label.created'
            }
        }],
        dataProvider: function(args) {
            // The UI framework, based on the `listView` id, injects the clicked
            // item in the `args.context`, for example:
            $.ajax({
                url: createURL('listCoffees'),
                data: {
                    id: args.context.coffee[0].id // list only for a particular resource by `id`
                },
                success: function(json) {
                    var items = json.listcoffeesresponse.coffee;
                    args.response.success({
                        data: items ? items[0] : {}
                    });
                } ,
                error:function(data) {
                    args.response.error(parseXMLHttpResponse(data));
                }
            });
        }

```

Tip: `$(window).trigger('cloudStack.fullRefresh')` can be used in any of the
dataProviders or action handlers to force UI item/view refresh.

UI plugin developer guide:
http://docs.cloudstack.apache.org/en/latest/developersguide/plugins.html#third-party-ui-plugins

## Exercises

1. Implement the Coffee UI plugin

2. Implement list and details views along with CRUD action for all the APIs

Challenge: Attempt and fix CloudStack UI issue(s)
https://github.com/apache/cloudstack/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3Aui
