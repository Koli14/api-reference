---
description: API reference of `@frontity/wp-source` package
---
# @frontity/wp-source

This package is in charge of getting the data from self-hosted WordPress or WordPress.com sites, and make it available from our React components.

## Table of Contents

<!-- toc -->

- [Installation](#installation)
- [Settings](#settings)
    + [`state.source.api`](#state-source-api)
    + [`state.source.subdirectory`](#state-source-subdirectory)
    + [`state.source.homepage`](#state-source-homepage)
    + [`state.source.postsPage`](#state-source-postspage)
    + [`state.source.categoryBase`](#state-source-categorybase)
    + [`state.source.tagBase`](#state-source-tagbase)
    + [`state.source.postEndpoint`](#state-source-postendpoint)
    + [`state.source.params`](#state-source-params)
    + [`state.source.postTypes`](#state-source-posttypes)
    + [`state.source.taxonomies`](#state-source-taxonomies)
- [How to use](#how-to-use)
- [API Reference](#api-reference)
  * [Actions](#actions)
    + [`actions.source.fetch()`](#actions-source-fetch)
  * [State](#state)
    + [`state.source.get()`](#state-source-get)
    + [`state.source[taxonomy][id]`](#state-source-taxonomy-id)
    + [`state.source[type][id]`](#state-source-type-id)
    + [`state.source.author[id]`](#state-source-author-id)
  * [Libraries](#libraries)
    + [`libraries.source.api.init()`](#libraries-source-api-init)
    + [`libraries.source.api.get()`](#libraries-source-api-get)
    + [`libraries.source.populate()`](#libraries-source-populate)
    + [`libraries.source.handlers`](#libraries-source-handlers)
    + [`libraries.source.redirections`](#libraries-source-redirections)
    + [`libraries.source.parse()`](#libraries-source-parse)
    + [`libraries.source.stringify()`](#libraries-source-stringify)
    + [`libraries.source.normalize()`](#libraries-source-normalize)

<!-- tocstop -->

## Installation

Add the `wp-source` package to your project:

```text
npm i @frontity/wp-source
```

{% hint style="info" %}
Both of the starter themes (`@frontity/mars-theme` & `@frontity/twentytwenty-theme`) available when doing `npx frontity create` already include this `wp-source` package
{% endhint %}

## Settings

This package needs to be included in your `frontity.settings.js` file as one of the packages that will be part of the Frontity project:

```javascript
module.exports = {
  packages: [
    "@frontity/mars-theme",
    "@frontity/tiny-router",
    {
      name: "@frontity/wp-source",
      state: {
        source: {
          api: "https://site.com/wp-json",
        },
      },
    },
  ],
};
```

These are the settings you can configure for this package in your `frontity.settings.js` file:

#### `state.source.api`<img src="https://img.shields.io/badge/REQUIRED-red.svg" >

The URL of your API. It can be from a self-hosted WordPress, like `https://site.com/wp-json` or from a WordPress.com site, like`https://public-api.wordpress.com/wp/v2/sites/site.wordpress.com`\(see [WordPress REST API on WordPress.com](https://developer.wordpress.com/2016/11/11/wordpress-rest-api-on-wordpress-com/)\).

Setting this value is the minimal configuration this package needs to work

#### `state.source.subdirectory`

A name or path indicating the subdirectory of your domain where your Frontity site lives. For example, if your site is in [https://mysite.com/blog](https://mysite.com/blog), you have to use it with the value of `blog` or `/blog`. It also transform links of the entities that come from the REST API.

#### `state.source.homepage`

This option allows you to show a specific page when accessing the homepage of your site. For example, if you set this value to `/about-us` then that page will be shown if you access `/`.

You have to configure your WordPress with the same setting.

![](../../.gitbook/assets/screen-shot-2019-08-30-at-13.08.35%20%283%29%20%281%29%20%282%29.png)

{% hint style="warning" %}
As this option overrides the `/` route, you should set `state.source.postsPage` as well in order to be able to access the posts archive in a different route.
{% endhint %}

#### `state.source.postsPage`

This option allows you to show the posts archive when accessing a specific URL of your site, instead of the homepage. For example, if you set this value to `/blog`, then the posts archive will be shown if you access `/blog` instead of `/`. It is useful when used in combination with `state.source.homepage`.

You have to configure your WordPress with the same setting.

![](../../.gitbook/assets/screen-shot-2019-08-30-at-13.08.35%20%283%29%20%282%29%20%282%29.png)

#### `state.source.categoryBase`

Change the base prefix of URLs for category pages with the indicated one.

{% hint style="warning" %}
For this option to work well, you have to put the same value in the WordPress site options.
{% endhint %}

#### `state.source.tagBase`

Change the base prefix of URLs for tag pages with the indicated one.

{% hint style="warning" %}
For this option to work well, you have to put the same value in the WordPress site options.
{% endhint %}

#### `state.source.postEndpoint`

Set the endpoint against which calls to the REST API are made **when posts are requested**, i.e. when fetching a single post, the post archive, date archives, categories, tags, authors, etc. This is useful when you want to use another post type as your default, for example “products”.

The default value is `"posts"`.

#### `state.source.params`

Object of params that will be used in every call to the WP REST API when using `actions.source.fetch`. This is useful to filter fields from the REST API, change the default `per_page` value and so on. For example, if you set this value to

```javascript
module.exports = {
  packages: [
    {
      name: "@frontity/wp-source",
      state: {
        source: {
          api: "https://site.com/wp-json",
          params: {
            per_page: 5,
            type: ["post", "page"],
          },
        },
      },
    },
  ],
};
```

and then you visit a URL \(or use `actions.source.fetch`\), the query part of the HTTP call to the REST API will be `per_page=5&type[]=post&type[]=page`.

#### `state.source.postTypes`

This option allows you to show the Custom Post Types you create at WordPress when accessing their URLs. It is an array of objects, each object being a different CPT. It has three arguments:

| Name | Type   | Required | Description | Example |
|------|--------|---------|----------|-------------|---------|
| **`type`**    | string | true     | The slug you configured for your Custom Post Type | `"movies"` |
| **`endpoint`** | string | true     | REST API endpoint from where this post type can be fetched. | `"movies"` | 
| `archive` | string | false     | the URL of the archive of this Custom Post Type, where all of them are listed. | `"/movies_archive"` | 


Differentiating `type` and `endpoint` may be confusing as they are usually the same. You can confirm you are doing it correctly going to the CPT `endpoint` :

![](../../.gitbook/assets/https___test_frontity_io__rest_route__wp_v2_movies%20%282%29%20%281%29%20%282%29.png)

So in this case, the settings would be:

```javascript
postTypes: [
  {
    type: "movies",
    endpoint: "movies",
    archive: "/movies_archive",
  },
];
```

#### `state.source.taxonomies`

Similar to `postTypes`setting, this one allows you to show the lists of posts of a Custom Taxonomies you create at WordPress when accessing their URLs. It is an array of objects, each object being a different Custom Taxonomy. It has four arguments:


| Name | Type | Required | Default Value | Description | Example |
|------|--------|---------|----------|-------------|---------|
| **`taxonomy`**    | `string` | `true` | | Taxonomy slug. The slug you configured for your Custom Taxonomy. | `"actors"` |
| **`endpoint`** | `string` | `true` | | REST API endpoint from where this post type can be fetched. | `"actor"` | 
| `postTypeEndpoint` | `string` | `false` | `posts` | REST API endpoint from which posts of this taxonomy can be fetched. If the Custom Taxonomy is meant to load Custom Post Types instead, you have to add its endpoint here. To clarify, although optional for posts in the case of a Custom Post Type this argument is **required**. | `"movies"` | 
| `params` | `object` | `false` |  | Extra params to be used while fetching the list of posts. | `{ per_page: 5, _embed: true }` | 


Again, differentiating `taxonomy` and `endpoint`may be confusing as they usually are the same too. You can confirm you are doing it correctly by going to the Custom Taxonomy `endpoint` :

![](../../.gitbook/assets/https___test_frontity_io__rest_route__wp_v2_actor%20%282%29%20%281%29%20%282%29.png)

Note that in this case `taxonomy`and `endpoint`are different. In the next example, we will fetch CPT "movies" instead of "posts", and add some params. It would be something like this:

```javascript
taxonomies: [
  {
    taxonomy: "actors",
    endpoint: "actor",
    postTypeEndpoint: "movies",
    params: {
      per_page: 5,
      _embed: true,
    },
  },
];
```


## How to use

This package will automatically retrieve data from the related WordPress routes when accesing a React route. 

The data got from WordPress REST API is organized and normalized in the state. This "normalization" of the data means the data is organized in the state in a way so there's no duplicated data in it and there's only one source of truth.

The state works with two main concepts: **links** and **entities**.

The state is designed so that you can know which entities correspond to which link, and then access the data of these entities in a simple way.

Because of this there's a 2 step process to get the information from a link:
1. Get the data related to the link 
1. Get the data related to the entities available in that link

{% hint style="warning" %}
For the data to exist, it will be necessary to request them previously using the `fetch` action.
{% endhint %}

```jsx
import React, { useEffect } from "react";
import { connect } from "frontity";

// In a React component that uses "connect":
const CategoryNature = ({ state, actions }) => {
  // 1. fetch data related to a path
  // With this useEffect we make the call to fetch
  // only the first time the component is rendered.
  // When the data is fetched, the state is updated with the new data
  // so the component is re-rendered and "data" will get proper content

  useEffect(() => {
    actions.source.fetch("/category/nature/");
  }, []);

  // 2. get data from frontity state
  const data = state.source.get("/category/nature/");

  // 3. get entities from frontity state
  if (data.isCategory) {
    // the category entity
    const category = state.source.category[data.id];

    // posts from that category
    const posts = data.items.map(({ type, id }) => state.source[type][id]);

    // 4. render!
    return (
      <>
        <h1>{category.name}</h1>
        {posts.map((p) => (
          <a href={p.link}>{p.title.rendered}</a>
        ))}
      </>
    );
  }

  return null;
};

export default connect(CategoryNature);
```

{% hint style="info" %}
If you want to know more about how to use the `wp-source` package, here you have some videos where Frontity DevRel team talks about it:

* 📺 [Frontity Talks 2020-01 - wp-source & CSS In JS \[1:36\]](https://www.youtube.com/watch?v=e-_66W8pfdY&t=96s)
* 📺 [Frontity Talks 2020-02 - Pagination example & wp-source \(state & fetch\) \[17:53\]](https://www.youtube.com/watch?v=eW5xZlpcqQk&t=1073s)
{% endhint %}

## API Reference

The [`wp-source` package](https://github.com/frontity/frontity/tree/dev/packages/wp-source) implements the [interface defined in the `source` package](https://github.com/frontity/frontity/blob/dev/packages/source/types.ts) and [adds some extra API](https://github.com/frontity/frontity/blob/dev/packages/wp-source/types.ts)



### Actions

Actions don't return data. Data is always accessed via the state. That's because Frontity is following the [Flux pattern](https://facebook.github.io/flux/) \(like Redux\).

{% hint style="info" %}
Read more about actions [here](../learning-frontity/actions.md)
{% endhint %}

#### `actions.source.fetch()`

This action fetches all entities related to a `link`, i.e. the pathname of a URL in your site.

> `(link: string, options: object) => Promise`

##### Parameters

| Name |  Property | Type | Required | Description | Example |
|------|--------|--------|---------|----------|----------|
| _**`[link]`**_    | |  `string` | `true` | Link representing a REST API endpoint or custom handler | `"/category/nature/"` |
| _`[options]`_ | | `object` | `false` | REST API endpoint from where this post type can be fetched. | `{ force: true }` |
| _`[options]`_ | `force` | `boolean` | - | The entities should be fetched again. | |

##### Return value

| Type | Description |
|--------|-------------|
| `Promise` | it doesn't return data but a promise that is resolved when the action is finished \(and state is updated\) |


All received data are populated in `state.source` and are accessible using the methods explained in the next section.

```javascript
actions.source.fetch("/category/nature/");
```

Even though actions don't return data, they return a promise that resolves when the action is finished.

So, you can do something like this:

```javascript
await actions.source.fetch("/some-post");
```

which is useful when you need to access the new state just after calling the action:

```javascript
await actions.source.fetch("/some-post"); // <- Wait until we fetch "/some-post".
const somePost = state.source.get("/some-post"); // <- The data will exist.
```

In React components, you won't need to use `async/await` with `fetch` because:

* [`useEffect` doesn't directly accept `async` functions](https://reactjs.org/docs/hooks-reference.html#useeffect) although [it can contain `async`](https://www.robinwieruch.de/react-hooks-fetch-data) functions
* They re-render when the `state` accessed changes.

```javascript
const SomePost = ({ actions, state }) => {
  useEffect(() => {
    // No need to use `async/await` here
    actions.source.fetch("/some-post");
  }, []);

  // The data will not exist at first, `dataPost.isReady` will be false.
  // But then, it will rerender when `actions.source.fetch` is finished.
  const dataPost = state.source.get("/some-post");

  // This will work just fine.
  return dataPost.isReady ? <Post link="/some-post" /> : <Loading />;
};
```

When `fetch` is called _again_ for the same `link` it does nothing, as all the entities have already been fetched and there is no need to request them again. If you do want to fetch them again, you can pass an options object to `source.fetch` with `force: true`:

```javascript
actions.source.fetch("/category/nature/", { force: true });
```

### State

#### `state.source.get()`

Returns an object that gives you info about the type of that link and related entities.

> `(link: string) => object`

##### Parameters

| Name | Type | Required | Description | Example |
|------|--------|---------|----------|----------|
| _**`[link]`**_  |  `string` | `true` | Link representing a REST API endpoint or custom handler | `"/category/nature/"` |

##### Return value

| Type | Description |
|--------|-------------|
| `object` | Info about the type of data represented in the URL |

For example:

```javascript
state.source.get("/category/nature/");
```

will return something like

```javascript
{
  // Entity properties.
  taxonomy: "category",
  id: 7,
  link: "/category/nature/page/3?s=park",
  query: {
    s: "park"
  },

  // Booleans that identify the type of link.
  isArchive: true,
  isCategory: true,
  isTaxonomy: true,

  // Booleans that show the fetch status.
  isFetching: false,
  isReady: true,

  // Archive properties.
  items: [{ type: "post", id: 60, link: "..." }, ...],
  total: 53,
  totalPages: 6,
  page: 3,
  route: "/category/nature",
  next: "/category/nature/page/4?s=park",
  previous: "/category/nature/page/2?s=park",

  // Search properties.
  isSearch: true,
  searchQuery: "park",
}
```

The information to distinguish each type of link is based on the [WP Template Hierarchy](https://wphierarchy.com/) and is as follows:

* archives: `isArchive`
  * taxonomy: `isTaxonomy`
    * category: `isCategory`
    * tag: `isTag`
    * deal: `isDeal`
  * author: `isAuthor`
  * postTypeArchive: `isPostTypeArchive`
    * post: `isHome`, `isPostArchive` \(`isFrontPage` optional\)
    * product: `isProductArchive`
  * date: `isDate`
* postTypes: `isPostType`
  * post: `isPost`
  * page: `isPage` \(`isFrontPage` optional\)
  * product: `isProduct`
  * media: `isMedia`, `isAttachment`
* 404: `is404`

Additionally, if calling `get()` has returned a status code higher than `400`, we add information about the error to the state. For example, if an error code was `500`, the state will include the following properties:

```javascript
{
  isError: true,
  is500: true,
  errorStatus: 500,
  errorStatusText: "Some string describing the error",

  // booleans that describe the fetch status
  isReady: true,
  isFetching: false
}
```

Properties added to each type are also based on the [WP REST API](https://developer.wordpress.org/rest-api/reference/):

* taxonomy: `taxonomy`, `id`
* author: `id`
* postTypeArchive: `type`
* date: `year`, `month`, `date`
* postType: `type`, `id`

#### `state.source[taxonomy][id]`

Access category, tag, or custom taxonomy’s entities. These entities have the same schema as specified in the [WP REST API](https://developer.wordpress.org/rest-api/reference/).

{% hint style="warning" %}
We are actually changing the WP REST API response, but **only for tags**, in which we are replacing the `taxonomy` value from `post_tag` to `tag`.
{% endhint %}

```text
source.category[2]
source.tag[13]
source.deal[3]
```

#### `state.source[type][id]`

Access posts, pages, attachments or custom post type’s entities. These entities have the same schema as specified in the [WP REST API](https://developer.wordpress.org/rest-api/reference/).

```text
source.post[60]
source.page[7]
source.product[36]
```

#### `state.source.author[id]`

Access author entities. These entities have the same schema as specified in the [WP REST API](https://developer.wordpress.org/rest-api/reference/).

```text
source.author[4]
```

### Libraries

#### `libraries.source.api.init()`

Set the URL to the WordPress REST API.

> `(options: object) => Promise`

##### Parameters

| Name |  Property | Type | Required | Description | Example |
|------|--------|--------|---------|----------|----------|
| _`[options]`_ | | `object` | `true` | options object | - |
| _`[options]`_ | `api` |`string` | `true` | URL pointing to a valid WP REST API. | `"https://test.frontity.io/wp-json"` |
| _`[options]`_ | `isWpCom` |`boolean` | `false` | if the WP REST route is from a WordPress.com hosted site. | `false` |


**Example**

```javascript
const { api } = libraries.source;

// for wp.org
api.init({
  api: "https://test.frontity.io/wp-json",
  isWpCom: false,
});

// for wp.com
api.init({
  api: "https://public-api.wordpress.com/wp/v2/sites/test.frontity.io",
  isWpCom: false,
});
```

#### `libraries.source.api.get()`

Request entity from the WordPress REST API.

> `(options: object) => Promise`

##### Parameters

| Name |  Property | Type | Required | Description | Example |
|------|--------|--------|---------|----------|----------|
| _`[options]`_ | | `object` | `true` | options object | |
| _`[options]`_ | `endpoint` | `string` | - | Name of the endpoint if is a `/wp/v2` endpoint \(e.g. `posts`\), or the full path of other REST endpoints \(e.g. `/acf/v3/posts`\). | `"posts"` |
| _`[options]`_ | `params` | `object` | - | Any parameter that will be included in the query params. | `{ _embed: true, categories: "2,3,4" }`|
| _`[options]`_ | `api` | `string` | - | Overrides the value set with `api.set.` | |
| _`[options]`_ | `isWpCom` | `boolean` | - | Overrides the value set with `api.set.` | |

##### Return value

| Type | Description |
|--------|-------------|
| `Promise` | it doesn't return data but a promise that is resolved when the action is finished \(and state is updated\) |


For more info, visit the [WP REST API reference](https://developer.wordpress.org/rest-api/reference).

**Example**

```javascript
const { api } = libraries.source;

// Get posts from categories 2, 3 and 4
const postsCategories = await api.get({
  endpoint: "posts",
  params: { _embed: true, categories: "2,3,4" },
});

// Get the page 14
const page14 = await api.get({
  endpoint: "pages",
  params: { _embed: true, include: "14" },
});

// Other endpoints:
const postBeautiesGullfoss = await api.get({
  endpoint: "/acf/v3/posts",
  params: { slug: "/the-beauties-of-gullfoss" },
});
```

#### `libraries.source.populate()`

Add entities to the Frontity state.

> `(options: object) => Promise`

##### Parameters

| Name |  Property | Type | Required | Default Value | Description | 
|------|--------|--------|---------|----------|----------|
| _`[options]`_ | | `object` | yes | | Options object |
| _`[options]`_ | **`response`** | `object` | yes | | The response object returned by `api.get().` |
| _`[options]`_ | **`state`** | `object` | yes | | The state object from the Frontity store. |
| _`[options]`_ | `subdirectory` | `string` | no | Value defined in `state.source.subdirectory` | Domain's subdirectory where your Frontity site is accessible. When this options is passed, this subdirectory is added to the entities' links. |
| _`[options]`_ | `force` | `boolean` | no | `false` | Value indicating if the entities should be overwritten |

##### Return value

| Type | Description |
|--------|-------------|
| `Array` | An array of objects with attributes `type`, `id` and `link` representing the added entities. |

Entities are normally never overwritten. So, if an entity already exists in the state and a new one is fetched, the one in the state will prevail. If you want to overwrite them, `populate` should be called with `force: true`.

**Example**

```javascript
const response = await libraries.source.api.get({ endpoint: "posts" })
const entitiesAdded = await libraries.source.populate({ response, state }))

entitiesAdded.forEach(({type, id, link}) => {
  console.log({type, id, link})
})
```

#### `libraries.source.handlers`

Handlers are objects that associate a path pattern with a function that gets the entities contained in that path. These `handlers` are used when `actions.source.fetch` is called.

A handler is defined by an object with the following properties:

| Name |  Type | Required | Description | 
|------|--------|---------|----------|----------|
| **`name`** | `string` | yes | Identifier of the handler. |
| **`priority`** | `number` | yes | Number that lets `fetch` to know in which order handlers should be evaluated. |
| **`pattern`** | `regExp` | yes | Pattern which paths are compared with. We use [path-to-regexp](https://github.com/pillarjs/path-to-regexp) under the hood, so check its documentation to know how to write patterns. |
| **`func`** | `function` | yes | Asynchronous function that retrieves entities and adds all info to the state. |

##### The `func` property

###### Arguments

The `func` property defined will receive an object with the following properties

| Name |  Type | Description | 
|------|--------|----------|----------|
| `link` | `string` | The link that are being fetched. |
| `params` | `string` | values obtained from the pattern after a match |
| `state` | `object` | Frontity state. |
| `libraries` | `object` | Frontity libraries. |
| `force` | `boolean` | If the entities should be fetched again. Internally, this parameter will be passed to the `actions.source.fetch` call. |

###### Return

| Type | Description |
|--------|-------------|
| `Promise` | Promise resolving to custom data

`libraries.source.handlers` is an array., so **to add new handlers we can use `libraries.source.handlers.push()`**

**Example**

```javascript
// A handler example to retrieve products
libraries.source.handlers.push({
  name: "product",
  priority: 10,
  pattern: "/product/:slug",
  func: async ({ link, params, state, libraries, force }) => {
    // 1. get product
    const response = await libraries.source.api.get({
      endpoint: "products",
      params: { slug: params.slug },
    });

    // 2. add product to state
    const [product] = await libraries.source.populate({
      response,
      state,
      force,
    });

    // 3. add link to data
    Object.assign(state.source.data[link], {
      id: product.id,
      type: product.type,
      isPostType: true,
      isProduct: true,
    });
  },
});
```

#### `libraries.source.redirections`

Redirections are objects that associate a path pattern with a function that returns a new path. These `redirections` are used when `actions.source.fetch` is executed, before `handlers`.

A redirection is defined by an object with the following properties:

| Name |  Type | Required | Description | 
|------|--------|---------|----------|
| **`name`** | `string` | yes | Identifier of the redirection. |
| **`priority`** | `number` | yes | Let `fetch` to know in which order redirections should be evaluated. |
| **`pattern`** | `regExp` | yes | Pattern which paths are compared with. We use [path-to-regexp](https://github.com/pillarjs/path-to-regexp) under the hood, so check its documentation to know how to write patterns. |
| **`func`** | `function` | yes | Function that returns a new path. It receives an object with the params obtained after a match. |

##### The `func` property

###### Arguments

The `func` property defined will receive an object with the following properties

| Name |  Type | Description | 
|------|--------|----------|----------|
| `slug` | `string` | The link that is being fetched. |

###### Return

| Type | Description |
|--------|-------------|
| `string` | a new path


**Example**

```javascript
// A redirection example to change tag base prefix
libraries.source.redirections.push({
  name: "tags",
  priority: 10,
  pattern: "/tag/:slug/",
  func: ({ slug }) => `/label/${slug}/`,
});
```

#### `libraries.source.parse()`

Utility for parsing links.

> `(link: string) => object`

##### Parameters

| Name |  Type | Required | Description | 
|------|--------|--------|----------|
| _`[link]`_ | `string` | yes |any link that points to entities in your site \(links, custom lists, etc.\) |

##### Return value

| Name |  Property | Type | Description | 
|------|--------|--------|---------|----------|----------|
| _`[resultParse]`_ | | `object` | Options object |
| _`[resultParse]`_ | `path` | `string` | Pathname without the page |
| _`[resultParse]`_ | `page` | `number` | The page number |
| _`[resultParse]`_ | `query` | `string` | Object with query parameters |
| _`[resultParse]`_ | `hash` | `string` | The hash value \(with `#`\). |


#### `libraries.source.stringify()`

Utility for building links from its attributes.

> `(args: object) => string`

##### Parameters

| Name |  Type | Required | Description | 
|------|--------|--------|----------|
| **`[path]`** | `string` | yes | pathname without the page |
| _`[page]`_ | `number` | no | The page number |
| _`[query]`_ | `object` | no | Object with query parameters |
| _`[hash]`_ | `string` | no | The hash value \(with `#`\). |

##### Return value

| Name |  Type | Description | 
|------|----------|----------|
| _`[link]`_ | `string` | Normalized link |

#### `libraries.source.normalize()`

> `(link: string) => string`

##### Parameters

| Name |  Type | Required | Description | 
|------|--------|--------|----------|
| **`[link]`** | `string` | yes | Any link that points to entities in your site \(links, custom lists, etc.\) |

##### Return value

| Name |  Type | Description | 
|------|----------|----------|
| _`[link]`_ | `string` | Normalized link |


{% hint style="info" %}
Still have questions? Ask [the community](https://community.frontity.org/)! We are here to help 😊
{% endhint %}














