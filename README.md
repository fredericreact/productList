# 1. Replace Redux with Context API (not great for high frequency updates)

# ContextAPI refresher

## Get state 

```javascript
import React, {useState} from 'react';
 
export const ProductsContext = React.createContext({products: []})
 
export default props => {
    const [productsList, setProductsList] = useState([
        {
            id: 'p1',
            title: 'Red Scarf',
            description: 'A pretty red scarf.',
            isFavorite: false
          },
          {
            id: 'p2',
            title: 'Blue T-Shirt',
            description: 'A pretty blue t-shirt.',
            isFavorite: false
          },
          {
            id: 'p3',
            title: 'Green Trousers',
            description: 'A pair of lightly green trousers.',
            isFavorite: false
          },
          {
            id: 'p4',
            title: 'Orange Hat',
            description: 'Street style! An orange hat.',
            isFavorite: false
          }
    ])
    return (
        <ProductsContext.Provider value={
            {products: productsList}
        }>
            {props.children}
        </ProductsContext.Provider>
    )
}

```

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
 
import './index.css';
import App from './App';
import ProductsProvider from './context/products-context';
 
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<ProductsProvider>
  <BrowserRouter>
    <App />
  </BrowserRouter>
</ProductsProvider>);

```

```javascript
import React, {useContext} from 'react';
 
import ProductItem from '../components/Products/ProductItem';
import {ProductsContext} from '../context/products-context'
import './Products.css';
 
const Products = props => {
  const productList = useContext(ProductsContext).products;
  return (
    <ul className="products-list">
      {productList.map(prod => (
        <ProductItem
          key={prod.id}
          id={prod.id}
          title={prod.title}
          description={prod.description}
          isFav={prod.isFavorite}
        />
      ))}
    </ul>
  );
};
 
export default Products;
 

```

## Update State

```javascript
import React, {useState} from 'react';
 
export const ProductsContext = React.createContext({
    products: [],
toggleFav: (id) =>{}
})
 
export default props => {
    const [productsList, setProductsList] = useState([
        {
            id: 'p1',
            title: 'Red Scarf',
            description: 'A pretty red scarf.',
            isFavorite: false
          },
          {
            id: 'p2',
            title: 'Blue T-Shirt',
            description: 'A pretty blue t-shirt.',
            isFavorite: false
          },
          {
            id: 'p3',
            title: 'Green Trousers',
            description: 'A pair of lightly green trousers.',
            isFavorite: false
          },
          {
            id: 'p4',
            title: 'Orange Hat',
            description: 'Street style! An orange hat.',
            isFavorite: false
          }
    ]);
 
    const toggleFavorite = productId => {
        setProductsList(currentProdList =>{
            const prodIndex = currentProdList.findIndex(
                p => p.id === productId
              );
              const newFavStatus = !currentProdList[prodIndex].isFavorite;
              const updatedProducts = [...currentProdList];
              updatedProducts[prodIndex] = {
                ...currentProdList[prodIndex],
                isFavorite: newFavStatus
              };
            return updatedProducts;
        })
    }
 
    return (
        <ProductsContext.Provider value={
            {products: productsList, toggleFav: toggleFavorite}
        }>
            {props.children}
        </ProductsContext.Provider>
    )
}

```

```javascript
import React, {useContext} from 'react';
 
import Card from '../UI/Card';
import './ProductItem.css';
import { ProductsContext } from '../../context/products-context';
 
const ProductItem = props => {
 
  const toggleFav = useContext(ProductsContext).toggleFav;
 
  const toggleFavHandler = () => {
    toggleFav(props.id)
  };
 
  return (
    <Card style={{ marginBottom: '1rem' }}>
      <div className="product-item">
        <h2 className={props.isFav ? 'is-fav' : ''}>{props.title}</h2>
        <p>{props.description}</p>
        <button
          className={!props.isFav ? 'button-outline' : ''}
          onClick={toggleFavHandler}
        >
          {props.isFav ? 'Un-Favorite' : 'Favorite'}
        </button>
      </div>
    </Card>
  );
};
 
export default ProductItem;
 

```
## Get state

```javascript
import React, {useContext} from 'react';
 
import FavoriteItem from '../components/Favorites/FavoriteItem';
import { ProductsContext } from '../context/products-context';
import './Products.css';
 
const Favorites = props => {
  const favoriteProducts = useContext(ProductsContext).products.filter(p=>p.isFavorite);
  let content = <p className="placeholder">Got no favorites yet!</p>;
  if (favoriteProducts.length > 0) {
    content = (
      <ul className="products-list">
        {favoriteProducts.map(prod => (
          <FavoriteItem
            key={prod.id}
            id={prod.id}
            title={prod.title}
            description={prod.description}
          />
        ))}
      </ul>
    );
  }
  return content;
};
 
export default Favorites;
 

```

# 2. Replace Redux with custom hooks (readme to be review, incomplete)

## set the store and custom hook

```javascript
import {useState, useEffect} from 'react';

let globalState = {};
let listeners = [];
let actions = {};

export const useStore = (shouldListen = true) => {
    const setState = useState(globalState)[1];

    const dispatch = (actionIdentifier,payload) => {
        const newState = actions[actionIdentifier](globalState, payload)
        globalState = {...globalState, ...newState}
        for (const listener of listeners) {
            listener(globalState);
        }
    }

    useEffect(()=>{
        if (shouldListen) {
            listeners.push(setState);
        }

        return ()=> {
            if (shouldListen) {
                listeners = listeners.filter(li => li!== setState)
            }
        }
    }, [setState, shouldListen]);


    return [globalState,dispatch];
}


export const initStore = (userActions, initialState) => {
    if (initialState) {
        globalState = {...globalState, ...initialState}
    }
    actions = {...actions, ...userActions}
}
```

```javascript
import {initStore} from './store'

const configureStore = () => {
    const actions = {
        TOGGLE_FAV: (curState, productId) => {

            const prodIndex = curState.products.findIndex(
                p => p.id === productId
              );
              const newFavStatus = !curState.products[prodIndex].isFavorite;
              const updatedProducts = [...curState.products];
              updatedProducts[prodIndex] = {
                ...curState.products[prodIndex],
                isFavorite: newFavStatus
              };

            return { products:updatedProducts}  

        }
    }
    initStore(actions, {products: [ 
        {
        id: 'p1',
        title: 'Red Scarf',
        description: 'A pretty red scarf.',
        isFavorite: false
      },
      {
        id: 'p2',
        title: 'Blue T-Shirt',
        description: 'A pretty blue t-shirt.',
        isFavorite: false
      },
      {
        id: 'p3',
        title: 'Green Trousers',
        description: 'A pair of lightly green trousers.',
        isFavorite: false
      },
      {
        id: 'p4',
        title: 'Orange Hat',
        description: 'Street style! An orange hat.',
        isFavorite: false
      }
    ]
})
}

export default configureStore;
```

## provide 


```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';

import './index.css';
import App from './App';
import configureProductsStore from './hooks-store/products-store';

configureProductsStore();

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

## use

```javascript
import React from 'react';
import { useStore } from '../../hooks-store/store';
import Card from '../UI/Card';
import './ProductItem.css';

const ProductItem = React.memo(props => {
  console.log('rendering')
  const dispatch = useStore(false)[1]

  const toggleFavHandler = () => {
    // toggleFav(props.id)
    dispatch('TOGGLE_FAV', props.id)
  };

  return (
    <Card style={{ marginBottom: '1rem' }}>
      <div className="product-item">
        <h2 className={props.isFav ? 'is-fav' : ''}>{props.title}</h2>
        <p>{props.description}</p>
        <button
          className={!props.isFav ? 'button-outline' : ''}
          onClick={toggleFavHandler}
        >
          {props.isFav ? 'Un-Favorite' : 'Favorite'}
        </button>
      </div>
    </Card>
  );
});

export default ProductItem;

```

```javascript
import React, {useContext} from 'react';

import FavoriteItem from '../components/Favorites/FavoriteItem';
import { useStore } from '../hooks-store/store';
import './Products.css';

const Favorites = props => {
  const state = useStore()[0];
  const favoriteProducts = state.products.filter(p => p.isFavorite)
  let content = <p className="placeholder">Got no favorites yet!</p>;
  if (favoriteProducts.length > 0) {
    content = (
      <ul className="products-list">
        {favoriteProducts.map(prod => (
          <FavoriteItem
            key={prod.id}
            id={prod.id}
            title={prod.title}
            description={prod.description}
          />
        ))}
      </ul>
    );
  }
  return content;
};

export default Favorites;

```


```javascript
import React, {useContext} from 'react';

import ProductItem from '../components/Products/ProductItem';
import { useStore } from '../hooks-store/store'
import './Products.css';

const Products = props => {
  const state = useStore()[0];
  return (
    <ul className="products-list">
      {state.products.map(prod => (
        <ProductItem
          key={prod.id}
          id={prod.id}
          title={prod.title}
          description={prod.description}
          isFav={prod.isFavorite}
        />
      ))}
    </ul>
  );
};

export default Products;

```

