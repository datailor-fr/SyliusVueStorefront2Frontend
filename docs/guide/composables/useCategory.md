# `useCategory`

## Features

`useCategory` composable is used for fetching categories by search parameters.

## API

* `categories: Category[]` - an array of fetched categories.
* `loading: boolean` - a reactive object containing information whether categories are loading.


### `search`
Function for fetching products based on passed `params`.


## Getters

* `getTree: AgnosticCategoryTree`

[AgnosticCategoryTree](https://docs.vuestorefront.io/v2/reference/api/core.agnosticcategorytree.html)

## Example

```js
import { onSSR } from '@vue-storefront/core';
import { computed } from '@vue/composition-api';
import { useCategory, categoryGetters } from '@realtainment/sylius';
export default {
  setup () {
    const { categories, search } = useCategory();

    onSSR(async () => {
      await search({ slug });
    });

    return {
      products,
      categoryGetters
    }
  }
};
```
