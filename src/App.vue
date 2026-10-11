<script setup>
import { computed, ref, watch } from 'vue'

const products = [
  { id: 1, name: 'Розовый ноутбук', price: 89990, category: 'Электроника', image: 'https://ir.ozone.ru/s3/multimedia-x/c1000/6591793785.jpg' },
  { id: 2, name: 'Красная механическая клавиатура', price: 7500, category: 'Электроника', image: 'https://cdn.prod.website-files.com/65f1868446603b58c3d61080/6934572207251499f3e3cb0f_Frame%202136139239.webp' },
  { id: 3, name: 'Кружка с медведем', price: 690, category: 'Аксессуары', image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQuP_1oaOXT9JPchKGDN47JMBXSEcJYRyFnUHO1orydwHCyaNE3tZS116w&s=10' },
  { id: 4, name: 'Красное худи с принтом', price: 3200, category: 'Одежда', image: 'https://nikifilini.com/wp-content/uploads/2025/05/zip_vizual-red-1-1.jpg' },
  { id: 5, name: 'Белая игровая мышь', price: 2400, category: 'Электроника', image: 'https://ainex.ru/upload/iblock/fc0/24gx5ft2fm1lkoqd7q62lla3qoupyj8b.jpg' },
  { id: 6, name: 'Наклейки с котами', price: 150, category: 'Аксессуары', image: 'https://ae-pic-a1.aliexpress-media.com/kf/S2a02a407cac7448fb662c47183948815I.jpg' },
  { id: 7, name: 'Изогнутый игровой монитор', price: 24990, category: 'Электроника', image: 'https://ae-pic-a1.aliexpress-media.com/kf/S6cfff7e0f84247e688e411ae7edefb735.jpg_480x480.jpg' },
  { id: 8, name: 'Кепка-аэродром', price: 1100, category: 'Одежда', image: 'https://basket-25.wbbasket.ru/vol4555/part455581/455581701/images/c516x688/1.webp' },
]

const search = ref('')
const category = ref('Все категории')
const sort = ref('')
const cart = ref(JSON.parse(localStorage.getItem('cart') || '[]').map(item => ({
  ...products.find(product => product.id === item.id),
  quantity: item.quantity,
})))
const promoCode = ref('')
const promoApplied = ref(false)
const promoMessage = ref('')

const filteredProducts = computed(() => {
  const result = products.filter(product =>
    product.name.toLowerCase().includes(search.value.toLowerCase().trim()) &&
    (category.value === 'Все категории' || product.category === category.value)
  )

  if (sort.value === 'asc') result.sort((a, b) => a.price - b.price)
  if (sort.value === 'desc') result.sort((a, b) => b.price - a.price)
  return result
})

const cartCount = computed(() => cart.value.reduce((sum, item) => sum + item.quantity, 0))
const total = computed(() => cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0) * (promoApplied.value ? 0.9 : 1))

watch(cart, value => localStorage.setItem('cart', JSON.stringify(value)), { deep: true })

function addToCart(product) {
  const item = cart.value.find(item => item.id === product.id)
  if (item) item.quantity++
  else cart.value.push({ ...product, quantity: 1 })
}

function removeFromCart(id) {
  cart.value = cart.value.filter(item => item.id !== id)
}

function clearCart() {
  cart.value = []
}

function applyPromo() {
  if (promoCode.value.trim().toUpperCase() === 'WEB') {
    promoApplied.value = true
    promoMessage.value = 'Скидка 10% применена'
  } else {
    promoApplied.value = false
    promoMessage.value = 'Неверный промокод'
  }
}

function rubles(price) {
  return new Intl.NumberFormat('ru-RU', { style: 'currency', currency: 'RUB', maximumFractionDigits: 0 }).format(price)
}
</script>

<template>
  <div class="page">
    <header class="header">
      <div>
        <p class="eyebrow">Вещи для работы и отдыха</p>
        <h1>Берлога</h1>
        <p class="intro">Кепка — на случай, если из берлоги всё-таки выйдешь.</p>
      </div>
      <aside id="cart" class="cart" aria-labelledby="cart-title">
        <div class="cart-heading">
          <div>
            <p class="eyebrow">Ваш заказ</p>
            <h2 id="cart-title">Корзина</h2>
          </div>
          <span class="cart-badge">{{ cartCount }}</span>
        </div>

        <p v-if="cart.length === 0" class="empty-cart">Пока пусто. Добавьте товар из каталога.</p>
        <template v-else>
          <ul class="cart-items">
            <li v-for="item in cart" :key="item.id" class="cart-item">
              <div class="cart-item-main">
                <strong>{{ item.name }}</strong>
                <small>{{ item.quantity }} × {{ rubles(item.price) }}</small>
                <small v-if="promoApplied" class="discount-price">Со скидкой: {{ rubles(item.price * 0.9) }} за шт.</small>
              </div>
              <div class="cart-item-side">
                <strong>{{ rubles(item.price * item.quantity * (promoApplied ? 0.9 : 1)) }}</strong>
                <button class="remove-button" type="button" :aria-label="`Удалить ${item.name}`" @click="removeFromCart(item.id)">Удалить</button>
              </div>
            </li>
          </ul>

          <div class="promo">
            <label for="promo-code">Промокод</label>
            <div class="promo-row">
              <input id="promo-code" v-model="promoCode" type="text" placeholder="Введите код" @keyup.enter="applyPromo" />
              <button type="button" @click="applyPromo">Применить</button>
            </div>
            <p v-if="promoMessage" class="promo-message" role="status">{{ promoMessage }}</p>
          </div>

          <div class="total"><span>Итого</span><strong>{{ rubles(total) }}</strong></div>
          <button class="clear-button" type="button" @click="clearCart">Очистить корзину</button>
        </template>
      </aside>
    </header>

    <main class="layout">
      <section class="catalog" aria-labelledby="catalog-title">
        <div class="section-heading">
          <div>
            <p class="eyebrow">Каталог</p>
            <h2 id="catalog-title">Наши товары</h2>
          </div>
          <span class="count">{{ filteredProducts.length }} из {{ products.length }}</span>
        </div>

        <div class="filters">
          <label>
            <span>Поиск</span>
            <input v-model="search" type="search" placeholder="Название товара" />
          </label>
          <label>
            <span>Категория</span>
            <select v-model="category">
              <option>Все категории</option>
              <option>Электроника</option>
              <option>Аксессуары</option>
              <option>Одежда</option>
            </select>
          </label>
          <label>
            <span>Цена</span>
            <select v-model="sort">
              <option value="">Без сортировки</option>
              <option value="asc">Сначала дешевле</option>
              <option value="desc">Сначала дороже</option>
            </select>
          </label>
        </div>

        <div v-if="filteredProducts.length" class="products">
          <article v-for="product in filteredProducts" :key="product.id" class="product-card">
            <img class="product-photo" :src="product.image" :alt="product.name" loading="lazy" referrerpolicy="no-referrer" />
            <div class="product-info">
              <p class="product-category">{{ product.category }}</p>
              <h3>{{ product.name }}</h3>
              <div class="product-bottom">
                <strong>{{ rubles(product.price) }}</strong>
                <button type="button" @click="addToCart(product)">В корзину</button>
              </div>
            </div>
          </article>
        </div>
        <p v-else class="empty-results">Товары не найдены. Попробуйте другой запрос.</p>
      </section>
    </main>
  </div>
</template>

<style>
* { box-sizing: border-box; }
body { margin: 0; background: #f6fbf4; color: #26342b; font-family: Arial, sans-serif; }
button, input, select { font: inherit; }
button { cursor: pointer; }
button:focus-visible, input:focus-visible, select:focus-visible, a:focus-visible { outline: 3px solid #427d5b; outline-offset: 2px; }
.page { max-width: 1240px; margin: auto; padding: 28px 24px 64px; }
.header { display: flex; justify-content: space-between; align-items: flex-start; gap: 20px; padding: 28px 32px; border-radius: 24px; background: #dcefdc; }
.eyebrow { margin: 0 0 8px; color: #4b7659; font-size: 12px; font-weight: 700; letter-spacing: .12em; text-transform: uppercase; }
h1, h2, h3, p { margin-top: 0; }
h1 { margin-bottom: 10px; font-size: clamp(32px, 4vw, 48px); }
h2 { margin-bottom: 0; font-size: 27px; }
h3 { margin: 0; font-size: 18px; line-height: 1.35; }
.intro { margin-bottom: 0; color: #5e6f63; }
.cart-badge { display: grid; place-items: center; min-width: 27px; height: 27px; padding: 0 5px; border-radius: 50%; background: #4f8e65; color: white; font-size: 13px; }
.layout { margin-top: 32px; }
.section-heading, .cart-heading { display: flex; justify-content: space-between; align-items: end; gap: 12px; margin-bottom: 20px; }
.count { color: #68766a; font-size: 14px; }
.filters { display: grid; grid-template-columns: 1.3fr 1fr 1fr; gap: 12px; margin-bottom: 20px; }
.filters label, .promo label { display: grid; gap: 7px; color: #536257; font-size: 13px; font-weight: 700; }
input, select { min-width: 0; width: 100%; height: 44px; padding: 0 12px; border: 1px solid #cfe3d1; border-radius: 10px; background: #fff; color: #26342b; }
.products { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 16px; }
.product-card { overflow: hidden; border: 1px solid #e0ece1; border-radius: 16px; background: #fff; }
.product-photo { display: block; width: 100%; aspect-ratio: 3 / 4; object-fit: contain; background-color: #e3f0df; }
.product-info { display: flex; flex-direction: column; min-height: 156px; padding: 16px; }
.product-category { margin-bottom: 8px; color: #56836a; font-size: 12px; }
.product-bottom { display: flex; align-items: center; justify-content: space-between; gap: 8px; margin-top: auto; padding-top: 16px; }
.product-bottom strong { white-space: nowrap; font-size: 16px; }
.product-bottom button, .promo-row button { border: 0; border-radius: 9px; background: #4f8e65; color: #fff; font-size: 13px; font-weight: 700; }
.product-bottom button { padding: 10px 12px; }
.product-bottom button:hover, .promo-row button:hover { background: #3e7152; }
.empty-results { padding: 35px 20px; border-radius: 16px; background: white; color: #5d6e61; text-align: center; }
.cart { width: 340px; flex: none; padding: 22px; border: 1px solid #d6e9d9; border-radius: 18px; background: #fff; }
.cart-heading { align-items: center; }
.empty-cart { margin: 26px 0 8px; color: #647568; line-height: 1.5; }
.cart-items { margin: 0; padding: 0; list-style: none; }
.cart-item { display: flex; justify-content: space-between; gap: 12px; padding: 14px 0; border-bottom: 1px solid #e5f0e6; }
.cart-item-main, .cart-item-side { display: flex; flex-direction: column; gap: 6px; }
.cart-item-main { min-width: 0; }
.cart-item-main strong { font-size: 14px; line-height: 1.3; }
.cart-item small { color: #6b7b6e; }
.cart-item .discount-price { color: #3f7955; }
.cart-item-side { flex: none; align-items: flex-end; text-align: right; }
.cart-item-side strong { font-size: 14px; }
.remove-button { padding: 0; border: 0; background: none; color: #3f7955; font-size: 12px; text-decoration: underline; }
.promo { margin-top: 23px; }
.promo-row { display: flex; gap: 7px; margin-top: 8px; }
.promo-row button { padding: 0 10px; }
.promo-message { margin: 8px 0 0; color: #3f7955; font-size: 12px; }
.total { display: flex; justify-content: space-between; gap: 10px; margin-top: 24px; padding-top: 18px; border-top: 1px solid #cfe5d3; font-size: 18px; }
.clear-button { width: 100%; margin-top: 16px; padding: 11px; border: 1px solid #a9cdb1; border-radius: 9px; background: #fff; color: #3f7955; font-size: 13px; font-weight: 700; }
.clear-button:hover { background: #f1f9f1; }
@media (max-width: 1000px) { .products { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
@media (max-width: 720px) { .page { padding: 14px 14px 40px; } .header { flex-direction: column; padding: 22px; } .cart { width: 100%; } .layout { margin-top: 28px; } .filters { grid-template-columns: 1fr 1fr; } .filters label:first-child { grid-column: 1 / -1; } .products { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
@media (max-width: 440px) { .product-info { min-height: 170px; padding: 12px; } .product-bottom { align-items: flex-start; flex-direction: column; } .product-bottom button { width: 100%; } h3 { font-size: 16px; } }
</style>
