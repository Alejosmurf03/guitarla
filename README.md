# GuitarLA — Tienda Online de Guitarras

Aplicación web de e-commerce para una tienda de guitarras, construida con **React 19** y **Vite**. El proyecto demuestra el uso de custom hooks, manejo de estado local, persistencia con `localStorage` y diseño responsive con Bootstrap.

---

## Tabla de Contenidos

- [Demo](#demo)
- [Características](#características)
- [Tecnologías](#tecnologías)
- [Arquitectura del Proyecto](#arquitectura-del-proyecto)
- [Estructura de Archivos](#estructura-de-archivos)
- [Custom Hook: useCart](#custom-hook-usecart)
- [Componentes](#componentes)
- [Persistencia con LocalStorage](#persistencia-con-localstorage)
- [Instalación y Uso](#instalación-y-uso)
- [Scripts Disponibles](#scripts-disponibles)
- [Decisiones Técnicas](#decisiones-técnicas)

---

## Demo

Puedes ver la aplicación desplegada en Netlify:
**[guitarla-react.netlify.app](https://guitarla-react.netlify.app)**

---

## Características

- Catálogo de **12 modelos de guitarras** con imagen, descripción y precio
- **Carrito de compras** interactivo con menú desplegable en el header
- **Agregar, eliminar y actualizar cantidad** de productos en el carrito
- Límite de cantidad por producto: mínimo 1, máximo 5 unidades
- **Cálculo automático del total** del carrito
- **Persistencia del carrito** entre sesiones usando `localStorage`
- Opción de **vaciar el carrito** completamente
- Diseño **responsive** adaptado a móvil, tablet y escritorio

---

## Tecnologías

| Tecnología | Versión | Rol |
|---|---|---|
| React | 19.2.5 | Librería UI principal |
| Vite | 8.0.10 | Build tool y servidor de desarrollo |
| Bootstrap | 5.2.3 | Framework CSS para estilos y layout |
| ESLint | 10.2.1 | Linting y calidad de código |
| JavaScript (ESM) | ES2020+ | Lenguaje principal (módulos ES nativos) |

**Fuentes:** Google Fonts — [Outfit](https://fonts.google.com/specimen/Outfit) (pesos 400, 700, 900)

---

## Arquitectura del Proyecto

La aplicación no utiliza librerías externas de manejo de estado (como Redux o Zustand). Todo el estado del carrito vive en un **custom hook (`useCart`)** que encapsula la lógica de negocio. El componente raíz `App` consume ese hook y distribuye las funciones necesarias via **prop drilling** a los componentes hijos.

```
App.jsx
  │
  ├── useCart()  ← custom hook (toda la lógica del carrito)
  │
  ├── <Header>   ← recibe: cart, cartTotal, isEmpty, removeFromCart,
  │                         decreaseQuantity, increaseQuantity, clearCart
  │
  └── <Guitar>[] ← recibe: guitar (datos), addToCart (función)
```

**Flujo de datos:**

1. `useCart` inicializa el carrito desde `localStorage` al montar la app
2. El usuario interactúa con `<Guitar>` → llama `addToCart`
3. `useCart` actualiza el estado del carrito
4. Un `useEffect` sincroniza el nuevo estado con `localStorage`
5. `<Header>` refleja los cambios en tiempo real vía props

---

## Estructura de Archivos

```
guitarla/
├── public/
│   └── img/                    # Imágenes estáticas (guitarras, logos, íconos)
├── src/
│   ├── components/
│   │   ├── Guitar.jsx          # Tarjeta de producto individual
│   │   └── Header.jsx          # Header con carrito desplegable
│   ├── data/
│   │   └── db.js               # Base de datos local (12 guitarras)
│   ├── hooks/
│   │   └── useCart.js          # Custom hook: toda la lógica del carrito
│   ├── App.jsx                 # Componente raíz
│   ├── index.css               # Estilos globales + customización Bootstrap
│   └── main.jsx                # Punto de entrada de React
├── index.html                  # HTML base (carga fuentes, monta #root)
├── vite.config.js              # Configuración de Vite
├── eslint.config.js            # Configuración de ESLint
└── package.json
```

---

## Custom Hook: useCart

`src/hooks/useCart.js` es el núcleo de la aplicación. Centraliza toda la lógica del carrito de compras para mantener los componentes limpios y enfocados en la UI.

```js
// Constantes de negocio
const MAX_QUANTITY = 5;
const MIN_QUANTITY = 1;

// Estado
const [cart, setCart] = useState(initialCart);

// Computed values (memoizados para evitar recálculos innecesarios)
const isEmpty   = useMemo(() => cart.length === 0, [cart]);
const cartTotal = useMemo(() => cart.reduce((total, item) => total + item.price * item.quantity, 0), [cart]);
```

### Funciones expuestas por el hook

| Función | Descripción |
|---|---|
| `addToCart(item)` | Agrega un producto al carrito. Si ya existe, incrementa la cantidad (respetando `MAX_QUANTITY`) |
| `removeFromCart(id)` | Elimina completamente un producto del carrito por su `id` |
| `increaseQuantity(id)` | Incrementa en 1 la cantidad de un producto (hasta `MAX_QUANTITY`) |
| `decreaseQuantity(id)` | Reduce en 1 la cantidad de un producto (hasta `MIN_QUANTITY`) |
| `clearCart()` | Vacía el carrito por completo |

### Hooks de React utilizados

- **`useState`** — Estado del carrito y de los datos de productos
- **`useEffect`** — Sincroniza el carrito con `localStorage` en cada cambio
- **`useMemo`** — Calcula `isEmpty` y `cartTotal` solo cuando el carrito cambia

---

## Componentes

### `<Header>`

Muestra el logo de la tienda y el ícono del carrito. Al hacer hover, despliega un panel con el detalle del carrito:

- Tabla con imagen, nombre, precio, controles de cantidad (+/−) y botón eliminar
- Total a pagar calculado automáticamente
- Botón para limpiar el carrito
- Mensaje de carrito vacío cuando no hay productos

**Props recibidas:**

```jsx
<Header
  cart={cart}
  cartTotal={cartTotal}
  isEmpty={isEmpty}
  removeFromCart={removeFromCart}
  decreaseQuantity={decreaseQuantity}
  increaseQuantity={increaseQuantity}
  clearCart={clearCart}
/>
```

### `<Guitar>`

Tarjeta de producto que muestra la imagen, nombre, descripción y precio de una guitarra, con un botón para agregarla al carrito.

**Props recibidas:**

```jsx
<Guitar
  guitar={guitar}       // objeto con id, name, image, description, price
  addToCart={addToCart} // función del custom hook
/>
```

---

## Persistencia con LocalStorage

El carrito se guarda automáticamente en `localStorage` cada vez que cambia, y se recupera al cargar la aplicación. Esto permite que el usuario no pierda su carrito al cerrar o refrescar el navegador.

```js
// Carga inicial desde localStorage
function initialCart() {
  const cartLocalStorage = localStorage.getItem('cart');
  return cartLocalStorage ? JSON.parse(cartLocalStorage) : [];
}

// Sincronización automática con useEffect
useEffect(() => {
  localStorage.setItem('cart', JSON.stringify(cart));
}, [cart]);
```

---

## Instalación y Uso

### Requisitos previos

- Node.js >= 18
- npm >= 9

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/Alejosmurf03/guitarla.git
cd guitarla

# 2. Instalar dependencias
npm install

# 3. Iniciar el servidor de desarrollo
npm run dev
```

La aplicación estará disponible en `http://localhost:5173`

---

## Scripts Disponibles

| Script | Descripción |
|---|---|
| `npm run dev` | Inicia el servidor de desarrollo con HMR (Hot Module Replacement) |
| `npm run build` | Genera la build de producción en la carpeta `dist/` |
| `npm run preview` | Sirve localmente la build de producción para revisión |
| `npm run lint` | Ejecuta ESLint para detectar errores de código |

---

## Decisiones Técnicas

**¿Por qué un Custom Hook en lugar de Context API?**
El carrito es el único estado global de esta aplicación y solo dos componentes lo consumen (`Header` y `Guitar`). Para este nivel de complejidad, un custom hook con prop drilling es más simple, más fácil de testear y evita el overhead de un Context Provider. Si la app creciera con más componentes anidados, migrar a Context sería el siguiente paso natural.

**¿Por qué `useMemo` en `isEmpty` y `cartTotal`?**
Ambos valores se derivan del estado `cart` y se usan en el render. Con `useMemo`, React solo los recalcula cuando `cart` cambia, evitando iteraciones innecesarias en cada re-render del componente padre.

**¿Por qué Vite en lugar de Create React App?**
Vite ofrece un servidor de desarrollo significativamente más rápido gracias a su uso de módulos ES nativos. También usa el compilador Oxc (a través de `@vitejs/plugin-react`), lo que acelera las transformaciones de JSX durante el desarrollo.

---

## Autor

Desarrollado por **Alejo** como proyecto de aprendizaje de React.
