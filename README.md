<div align="center"> 

# Projeto - Aplicação Instadev Mobile React Native

</div>

<br>

<div align="center">

[![Generic badge](https://img.shields.io/badge/Made%20by-Renan%20Borba-purple.svg)](https://shields.io/) [![Build Status](https://img.shields.io/github/stars/RenanBorba/react-native-instadev.svg)](https://github.com/RenanBorba/react-native-instadev) [![Build Status](https://img.shields.io/github/forks/RenanBorba/react-native-instadev.svg)](https://github.com/RenanBorba/react-native-instadev) [![made-for-VSCode](https://img.shields.io/badge/Made%20for-VSCode-1f425f.svg)](https://code.visualstudio.com/) [![npm version](https://badge.fury.io/js/react-native.svg)](https://badge.fury.io/js/react-native) [![Open Source Love svg2](https://badges.frapsoft.com/os/v2/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)

<br>

![insta](https://user-images.githubusercontent.com/48495838/85884098-71d6f600-b7b8-11ea-8fe8-4928f9059f23.png)

</div>

<br>

Aplicação Front-end Mobile desenvolvida em React Native para clone do Feed do app Instagram com Scroll infinito (rolagem itens), voltada para devs e permite a interação entre os desenvolvedores.

<br><br>

<div align="center">

![instadev](https://user-images.githubusercontent.com/48495838/84696821-74695e00-af23-11ea-8cfd-a9545fabad99.png)

</div>

<br><br>

## :rocket: Tecnologias
<ul>
  <li>Expo</li>
  <li>Components</li>
  <li>Routes</li>
  <li>react-navigation</li>
  <li>react-native-gesture-handler</li>
  <li>react-native-reanimated</li>
  <li>json-server (Server API fake)</li>
  <li>FlatList</li>
  <li>useState</li>
  <li>useEffect</li>
  <li>useCallback</li>
  <li>styled-components</li>
  <li>Animated</li>
</ul>

<br><br>

## :arrow_forward: Start
<ul>
  <li>npm install</li>
  <li>npm run start / npm start</li>
</ul>

<br><br>

## :punch: Como contribuir
<ul>
  <li>Dê um fork nesse repositório</li>
  <li>Crie a sua branch com a feature</li>
    <ul>
      <li>git checkout -b my-feature</li>
    </ul>
  <li>Commit a sua contribuição</li>
    <ul>
      <li>git commit -m 'feat: My feature'</li>
    </ul>
  <li>Push a sua branch</li>
    <ul>
      <li>git push origin my-feature</li>
    </ul>
</ul>

<br><br>
<br>

## :mega: Segue abaixo as principais estruturas e interface principal:

<br><br><br>

## src/pages/Feed/index.js
```js
import React, { useState, useEffect, useCallback } from 'react';
import { View, FlatList } from 'react-native';

import LazyImage from '../../components/LazyImage'
import
  {
    Post,
    Header,
    Avatar, Name,
    Description,
    Loading
  } from './styles';

export default function Feed() {
const [feed, setFeed] = useState([]);
const [page, setPage] = useState(1);
const [total, setTotal] = useState(0);
const [loading, setLoading] = useState(false);
const [refreshing, setRefreshing] = useState(false);
const [viewable, setViewable] = useState([]);

async function loadPage(pageNumber = page, shouldRefresh = false) {
  if (total && pageNumber > total) return;

  setLoading(true);

  const response = await fetch (
    `http://localhost:3000/feed?_expand=author&_limit=5&_page=${pageNumber}`
  );

  const data = await response.json();
  const totalItems = response.headers.get('X-Total-Count');

  // Arredondar númeto total de itens
  setTotal(Math.floor(totalItems / 5));
  setFeed(shouldRefresh ? data : [...feed, ...data]);
  setPage(pageNumber + 1);
  setLoading(false);
}

  useEffect(() => {
    loadPage();
  }, []);

  async function refreshList() {
    setRefreshing(true);

    await loadPage(1, true);

    setRefreshing(false);
  }

  const handleViewableChanged = useCallback(({ changed }) => {
    setViewable(changed.map(({ item }) => item.id));
  }, []);

  return (
    <View>
      <FlatList
        data={ feed }
        keyExtractor={post => String(post.id)}
        onEndReached={() => loadPage()}
        // 10%
        onEndReachedThreshold={10}
        // Pull-to-refresh
        onRefresh={ refreshList }
        refreshing={ refreshing }
        onViewableItensChanged={ handleViewableChanged }
        viewabilityConfig={{ viewAreaCoveragePercentThreshold : 20 }}
        ListFooterComponent={ loading && <Loading /> }
        renderItem={({ item }) => (
          <Post>
            <Header>
              <Avatar source={{ uri: item.author.avatar }} />
              <Name>{ item.author.name }</Name>
            </Header>

            <LazyImage
              shouldLoad={ viewable.includes(item.id) }
              aspectRatio={ item.aspectRatio }
              smallSource={{ uri: item.small }}
              source={{ uri: item.image }}
            />

            <Description>
              <Name>{ item.author.name }</Name> { item.description }
            </Description>
          </Post>
        )}
      />
    </View>
  );
};
```

<br><br>

## Scroll List, Exemplo 1

![1](https://user-images.githubusercontent.com/48495838/67954358-96e67b00-fbcf-11e9-829f-00478bc942ef.JPG)

<br><br>

![2](https://user-images.githubusercontent.com/48495838/67954359-977f1180-fbcf-11e9-823d-b90ab12cb8d8.JPG)

<br><br>

## Scroll List, Exemplo 2

![3](https://user-images.githubusercontent.com/48495838/67954360-977f1180-fbcf-11e9-8688-a1d9a6f695fa.JPG)

<br><br><br><br>

## src/components/LazyImage/index.js
```js
import React, { useState, useEffect } from 'react';
import { Animated } from 'react-native';

import { Small, Original } from './styles';

const OriginalAnimated = Animated.createAnimatedComponent(Original);

export default function LazyImage({
  smallSource,
  source,
  aspectRatio,
  shouldLoad
}) {
  const opacity = new Animated.Value(0);
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    if (shouldLoad == true) {
      //setTimeout(() => {
      setLoaded(true);
  //}, 1000);
    }
  }, [shouldLoad]);

  function handleAnimate() {
    Animated.timing(opacity, {
      toValue: 1,
      duration: 500,
      useNativeDriver: true
    }).start();
  }

  return (
    <Small
      source={ smallSource }
      ratio={ aspectRatio }
      resizeMode="contain"
      blurRadius={1.8}
    >
      { loaded && <OriginalAnimated
        style={{ opacity }}
        source={ source }
        ratio={ aspectRatio }
        resizeMode="contain"
        onLoadEnd={ handleAnimate }
      /> }
    </Small>
  );
};
```

<br><br>

## Efeito Blur

![0](https://user-images.githubusercontent.com/48495838/67954355-964de480-fbcf-11e9-9d82-0a06d0b55b83.jpg)
