# Entity Query API Deep Dive

This is an overview of what I'll be covering during the 2016-04-26 DBUG meetup. We'll be using the [Entity Query API](https://www.drupal.org/project/entityqueryapi) module as a reference for some "in-depth" looks at custom module development in Drupal 8.

# Why?
- Much of Drupal 8's out-of-the-box RESTful tools are fantastic, but in a headless or API heavy site they don't get you "all the way there."
- Notably, one can't query for a collection of entities without creating a custom View.
  - E.G. `GET /node` is a 404, where most APIs would return _all_ nodes and allow you to filter them.
  - Although Views is in core, it's not really an ideal solution.
    - Views forces you to expose every filter that you would like to allow as an exposed filter, rather than just letting the client specify the filters they need.
    - At present, it's impossible to use alternative authentication provides in the same way you might allow "basic_auth" on the RESTful config page.
  - Forcing you to create a view for every possible collection endpoint slows development time
    - It also may not be what you want in a public API, which should allow the consumer of the API to get the data they need, however they want it.

# What is it?
- Entity Query API is a module allows one to expose entities arbitrary queries by wrapping using Drupal Core's Entity [QueryInterface](https://api.drupal.org/QueryInterface).
- By standardizing on a URI syntax for sorting, paging, and filtering we can start to develop tooling which speeds development, but also fills the gap in core's RESTful services.
- If you'd like to help develop dapi.js, a JS library I'm imagining that will provide a low-level, universal interface to Drupal's entity system over REST, contact me! [gabe@aten.io](mailto:gabe@aten.io)

# How does it work?
This is just a roadmap for what I'd like to cover, if you think we're not going to get to something you want to see, speak up and we'll talk about that. All paths are relative to the root of the entityqueryapi module.

- `entityqueryapi.info.yml`
  - `configure` key
  - [Documentation](https://www.drupal.org/node/2000204)
- `entityqueryapi.routing.yml`
  - named routes
  - `_controller`
    - multiple methods
  - `requirements`
    - `_custom_access`
    - `_permission`
  - path parameters
    - Custom parameter upcasting using services
    - See: `src/EntityTypeParamConverter.php`
    - [Documentation](https://www.drupal.org/node/2122223)
  - [Documentation](https://www.drupal.org/node/2092643)
- `entityqueryapi.services.yml`
  - What are services?
  - `arguments`
  - `tags`
  - Why create them?
    - Helps you think about modularity
    - Exposese your code for use by other modules
      - Let's other modules overrided them
- Dependency Injection
  - What is it: A fancy name for black magic.
  - How to do it:
    - In Contollers
      - Via a `create` method
      - See: `src/Contoller/EntityQueryApiController.php`
    - In Services
      - In `<modulename>.services.yml`
      - See: `src/QueryBuilder/QueryBuilder.php`
  - Why?
    - Unit testing
      - Which I have no examples of :sorrow:
      - Not imperitive
    - Persitent services
    - Just super handy
    - Good example: `src/Controller/EntityQueryApiController.php:53:7`
    - Bad example: `src/Controller/EntityQueryApiController.php:66:17`
  - [Documentation](https://www.drupal.org/node/2133171)
- Type hinting
  - What, why, how.
- Creating and using custom configuration
  - Mutable and imutable configuration
    - See: `src/Controller/ConfigContoller.php:45:7`
  - Creating/Updating
    - Really easy
    - See: `src/Controller/ConfigContoller.php:208:7`
  - Accessing
    - See: `src/Controller/ConfigContoller.php:205:16`
    - See: `src/Controller/ConfigContoller.php:85:16`
  - Overriding
    - `settings.php`
      - `$config[<name>][<subvalue>] = <override>;`
    - [Documentation](https://www.drupal.org/node/1928898)

# Misc.
- [Or Group](http://module.dev/entity/node?_format=json&range[length]=1&sort_0[field]=created&sort_0[direction]=ASC&group_0[conjunction]=OR&condition_0[field]=type&condition_0[value]=foo&condition_0[operator]=EQ&condition_0[group]=group_0&condition_1[field]=promote&condition_1[value]=1&condition_1[operator]=EQ&condition_1[group]=group_0)
