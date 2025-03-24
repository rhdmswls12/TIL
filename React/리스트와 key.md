## 리스트와 key

<br />

## 리스트 항목 컴포넌트 추출하기

`RecipeList` 컴포넌트에는 두 개의 중첩된 `map` 호출이 포함되어 있습니다. 이를 단순화하기 위해 `id`, `name`, `ingredients` props를 허용하는 `Recipe` 컴포넌트를 추출합니다. 외부 `key`를 어디에 위치하고 그 이유는 무엇일까요?

```jsx
// data.js
export const recipes = [
  {
    id: "greek-salad",
    name: "Greek Salad",
    ingredients: ["tomatoes", "cucumber", "onion", "olives", "feta"],
  },
  {
    id: "hawaiian-pizza",
    name: "Hawaiian Pizza",
    ingredients: [
      "pizza crust",
      "pizza sauce",
      "mozzarella",
      "ham",
      "pineapple",
    ],
  },
  {
    id: "hummus",
    name: "Hummus",
    ingredients: ["chickpeas", "olive oil", "garlic cloves", "lemon", "tahini"],
  },
];
```

```jsx
// App.js
import { recipes } from "./data.js";

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map((recipe) => (
        <div key={recipe.id}>
          <h2>{recipe.name}</h2>
          <ul>
            {recipe.ingredients.map((ingredient) => (
              <li key={ingredient}>{ingredient}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

<br />

## 풀이

```jsx
import { recipes } from "./data.js";

function Recipe({ id, name, ingredients }) {
  return (
    <div>
      <h2>{name}</h2>
      <ul>
        {ingredients.map((ingredient) => (
          <li key={ingredient}>{ingredient}</li>
        ))}
      </ul>
    </div>
  );
}

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map((recipe) => (
        <Recipe {...recipe} key={recipe.id} />
      ))}
    </div>
  );
}
```

- `{recipes.map(recipe =>
    <Recipe {...recipe} key={recipe.id} />
)}`
  위와 같이 key를 `<Recipe />`에 지정하고 있다.

<br />

## key가 Recipe 바깥에 있어야 하는 이유

```jsx
{
  recipes.map(
    (recipe) => <Recipe {...recipe} key={recipe.id} /> // key가 여기 있어야 함
  );
}
```

- 리액트는 `Recipe` 컴포넌트가 리스트의 개별 요소임을 인식할 수 있다.
- 리스트가 업데이트될 때 리액트가 변경된 항목을 정확하게 식별할 수 있다.
- 렌더링 최적화가 가능해지고 불필요한 리렌더링을 방지할 수 있다.

<br />

## key 위치에 따른 React의 리스트 관리 방식

```jsx
{
  recipes.map((recipe) => (
    <div key={recipe.id}>
      {" "}
      {/* key는 div에 위치 */}
      <h2>{recipe.name}</h2>
      <ul>
        {recipe.ingredients.map((ingredient) => (
          <li key={ingredient}>{ingredient}</li>
        ))}
      </ul>
    </div>
  ));
}
```

- 이전 코드에서는 key가 `<div>`에 위치하고 있다. 즉 `recipes` 배열이 변경될 때 리액트는 `<div>` 요소들을 기준으로 리스트의 변경을 추적한다.

```jsx
{
  recipes.map(
    (recipe) => <Recipe {...recipe} key={recipe.id} /> // key는 Recipe 컴포넌트 자체에 위치
  );
}
```

- 이제 key가 `<div>`가 아닌 `<Recipe />`컴포넌트 자체에 위치하고 있다. 따라서 리액트가 직접 `<Recipe />` 컴포넌트 배열을 관리한다.

<br />

<aside>
💡

- key는 리액트가 리스트의 각 항목을 추적하는 데 필요하다.
- key는 리스트의 개별 요소가 있는 최상위 컴포넌트에 두어야 한다.
- `Recipe` 컴포넌트를 분리했으므로 key를 `Recipe`에 지정해야 리액트가 `Recipe` 컴포넌트가 리스트 요소임을 정확히 이해할 수 있다.
</aside>
