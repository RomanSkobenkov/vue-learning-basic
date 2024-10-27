# Урок 2

*Императивный код* - код, в котором описываются действия до достижения результата (сами готовим яичницу - купить яйца → включить плиту → …).

*Декларативный код* - описываем нужный результат (приходим в ресторан и просто говорим, что нам нужна яичница).

---

В `v-for` надо быть аккуратным, если мы работаем не с объектами. И к `v-model` тогда надо привязывать значение по ключу массива (т.к. простые значения передаются по значению, а не по ссылке). Пример:

```jsx
// guests = string[]
<div v-for="guest,i in guests" class="form-group">
	<input v-model.trim="guests[i]" type="text" class="form-control" >
</div>
```

---

# Хуки жизненного цикла

!https://vuejs.org/assets/lifecycle.MuZLBFAS.png

## `beforeCreate` и `beforeMount`

Используются редко.

## `created`

Здесь объект копируется на верхний уровень, появляется реактивность, компонент готов к работе. Сюда часто пишут запросы на сервер для получения данных. При этом рендеринг не будет остановлен, т.к. ни один хук не тормозит рендеринг, это просто момент запуска.

## `mounted`

Здесь часто дорабатывают поведение готовой разметки. Например фокус на поле. Или использование сторонних библиотек (highlight.js например).

## `updated`

Очень страшная вещь. Будет вызываться например на любой инпут любого поля, на любое изменение всего.

---

# `$nextTick`

Проблема:

Допустим есть код:

```jsx
methods: {
	addGuest(){
		this.guests.push('');
		this.$refs.guestInps[this.$refs.guestInps.length - 1].focus();
	},
},
```

При такой реализации `focus()` будет отрабатывать некорректно. Причина в том, что Vue не перерендеривает шаблон на каждой операции с данными. Он сначала проводит все рассчёты из методов. И здесь он вызовет `focus()` в момент, когда элемента ещё не существует.

`$nextTick` создан для решения этой проблемы: он вызывается после всех расчётов и перерендера. Т.е. правильная реализация будет такой:

```jsx
methods: {
	addGuest(){
		this.guests.push('');

		this.$nextTick(() => {
			this.$refs.guestInps[this.$refs.guestInps.length - 1].focus();
		});
	},
},
```

Здесь вместо `$nextTick` можно было бы написать и `setTimeout`, кстати, но это костыльное решение для того, что должен делать именно `$nextTick`.

`$nextTick` это что-то вроде `addEventListener` на завершение рендера шаблона.

---

# `:classes`

Пример получения нужного класса через `computed`:

```jsx
<button :disabled="!userValid" :class="sendBtnClasses" class="btn">Send Data</button>
...
...
...
userValid(){
	return Object.values(this.user).every(val => val.length > 0);
},
sendBtnClasses(){
	return {
		'btn-danger': !this.userValid,
		'btn-success': this.userValid
	}
},
```

Примерчик задания bgc в зависимости от длины строки:

```jsx

<label>Password</label>
<input type="password" class="form-control" v-model.trim="user.password">
<div class="passwordGuarded" :style="passwordColor"></div>
...
...
...
computed: {
	passwordColor(){
		let q = Math.min(this.user.password.length / 8, 1);
		let color = `rgb(${(1 - q) * 255}, ${q * 255}, 0)`;

		return {
			backgroundColor: color
		}
	}
},
```

---

# Checkboxes and Radio

В случае с чекбоксами и радио инпутами `v-model` распадается на проверку сравнения значения с текущим в `checked`. **Поэтому `value` мы в таких инпутах не переписываем.**

Пример radio:

```jsx
<div class="form-group mb-3">
	Sex:
	<div class="form-check">
		<label class="form-check-label">
			<input v-model="sex" value="0" class="form-check-input" type="radio" name="sex">Men
		</label>
	</div>
	<div class="form-check">
		<label class="form-check-label">
			<!-- :checked="'sex' == thisInput.value" @change="if(e.t.checked){ sex = thisInput.value }" -->
			<input v-model="sex" value="1" class="form-check-input" type="radio" name="sex">Women
		</label>
	</div>
</div>
...
...
...
data: function(){
	return {
		sex: 0,
},
```

Пример чекбоксы:

```jsx
<div>
	<h3>Options</h3>
	<div v-for="item,name in wishlist" class="form-group">
		<label class="form-check-label">
			<input v-model="wishlist[name]" class="form-check-input" 
			type="checkbox" :name="name">
			{{ name }}
		</label>
	</div>
</div>
...
...
...
data: function(){
	return {
		wishlist: {
			allIncluded: false,
			firstLine: false
		},
	}
},
```