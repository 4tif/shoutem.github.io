---
layout: doc
permalink: /docs/extensions/my-first-extension/working-with-data
title: Working with data
section: My first extension
---

# Working with data
<hr />

Let's fetch data from the Shoutem Cloud storage to the app. First, remove the assets folder. We don't need it anymore. Create `app/reducer.js` file which will contain `reducer` defining initial app state and how the state changes.

Package [@shoutem/redux-io](https://github.com/shoutem/redux-io) has `reducers` and `actions` that communicate with Shoutem CMS. Reducer `storage` retrieves data (e.g. restaurants) in a dictionary while `collection` stores data ID's in an array to persist its order.

```javascript{1-9}
#file: app/reducer.js
import { storage, collection } from '@shoutem/redux-io';
import { combineReducers } from 'redux';
import { ext } from './extension';

// combine reducers into one root reducer
export default combineReducers({
  restaurants: storage(ext('Restaurants')),
  allRestaurants: collection(ext('Restaurants'), 'all')
});
```

We've used `ext` function to get full schema name (`{{ site.example.devName }}.restaurants.Restaurants`). Root reducer needs to be exported from `app/index.js` as `reducer`, so app can find it:

```javascript{4,9}
#file: app/index.js
// Reference for app/index.js can be found here:
// http://shoutem.github.io/docs/extensions/reference/extension-exports

import reducer from './reducer';
import * as extension from './extension.js';

export const screens = extension.screens;

export { reducer };
```

Find more information about extension parts [here]({{ site.url }}/docs/extensions/reference/extension-exports).

We will fetch restaurants from **Shoutem Cloud Storage** in `List` screen with `find` action creator. Also, we'll use 3 helper functions from the `@shoutem/redux-io` package:
 
 - `isBusy` - is data being fetched,
 - `shouldRefresh` - should data be (re)fetched,
 - `getCollection` - merges `storage` dictionary and `collection` ID array into an `array` of objects.

```javascript{1-6}
#file: app/screens/List.js
import {
  find,
  isBusy,
  shouldRefresh,
  getCollection
} from '@shoutem/redux-io';
```

The complete code is for `app/screens/List.js` is available below.

Fetch data in `componentDidMount` lifecycle method.

```javascript{2-9}
#file: app/screens/List.js
export class List extends Component {
  componentDidMount() {
    const { find, restaurants } = this.props;
    
    if (shouldRefresh(restaurants)) {
      find(ext('Restaurants'), 'all', {
          include: 'image',
      })
    }
  }
  ...
}
```

Implement rendering with fetched data.

```JSX{2,8-9}
#file: app/screens/List.js
  render() {
    const { restaurants } = this.props;
    
    return (
      <Screen>
        <NavigationBar title="RESTAURANTS" />
        <ListView
          data={restaurants}
          loading={isBusy(restaurants)}
          renderRow={restaurant => this.renderRow(restaurant)}
        />
      </Screen>
    );
  }
```

Once fetched, restaurants will get to the app state. Convert them to an array format with `getCollection` function and connect `find` to redux store.

```javascript{2-6}
#file: app/screens/List.js
export default connect(
  (state) => ({
    // get an array of restaurants from allRestaurants collection
    restaurants: getCollection(state[ext()].allRestaurants, state)
  }),
  { navigateTo, find }
)(List);
```

This is the final result of `List` screen:

```JSX
#file: app/screens/List.js
import React, {
  Component
} from 'react';

import {
  TouchableOpacity,
} from 'react-native';

import {
  Image,
  ListView,
  Tile,
  Title,
  Subtitle,
  Overlay,
  Screen
} from '@shoutem/ui';

import { NavigationBar } from '@shoutem/ui/navigation';
import { navigateTo } from '@shoutem/core/navigation';
import { ext } from '../extension';
import { connect } from 'react-redux';

import {
  find,
  isBusy,
  shouldRefresh,
  getCollection
} from '@shoutem/redux-io';

export class List extends Component {
  constructor(props) {
    super(props);

    // bind renderRow function to get the correct props
    this.renderRow = this.renderRow.bind(this);
  }

  componentDidMount() {
    const { find, restaurants } = this.props;
    
    if (shouldRefresh(restaurants)) {
      find(ext('Restaurants'), 'all', {
          include: 'image',
      })
    }
  }

  // defines the UI of each row in the list
  renderRow(restaurant) {
    const { navigateTo } = this.props;

    return (
      <TouchableOpacity onPress={() => navigateTo({
        screen: ext('Details'),
        props: { restaurant }
      })}>
        <Image styleName="large-banner" source={% raw %}{{ uri: restaurant.image &&
        restaurant.image.url ? restaurant.image.url : undefined }}{% endraw %}>
          <Tile>
            <Title>{restaurant.name}</Title>
            <Subtitle>{restaurant.address}</Subtitle>
          </Tile>
        </Image>
      </TouchableOpacity>
    );
  }

  render() {
    const { restaurants } = this.props;
    
    return (
      <Screen>
        <NavigationBar title="RESTAURANTS" />
        <ListView
          data={restaurants}
          loading={isBusy(restaurants)}
          renderRow={restaurant => this.renderRow(restaurant)}
        />
      </Screen>
    );
  }
}

// connect screen to redux store
export default connect(
  (state) => ({
    // get an array of restaurants from allRestaurants collection
    restaurants: getCollection(state[ext()].allRestaurants, state)
  }),
  { navigateTo, find }
)(List);
```

Let's check how it works:

```ShellSession
$ shoutem push
Uploading `Restaurants` extension to Shoutem...
Success!
```

Everything should work as a charm!

<p class="image">
<img src='{{ site.url }}/img/my-first-extension/working-with-data.png'/>
</p>

That's it! You've done it. You just made your first extension using **Shoutem UI Toolkit** and **Shoutem Cloud Storage**. Great job!
