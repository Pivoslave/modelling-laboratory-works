## Комп'ютерні системи імітаційного моделювання
## СПм-25-1, **Царенко Денис Олегович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 8, модель у середовищі NetLogo:
[Segregation Simple Extension 1](https://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/IABM%20Textbook/chapter%203/Segregation%20Extensions/Segregation%20Simple%20Extension%201.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Додано алгоритм <pre>check-disappearance</pre>, що вираховує шанс зникнення черепахи**, на який впливає кількість сусідів іншого кольору (вага кожного такого сусіда - 4%) та сумарний процент таких же сусідів (вага - процент однакових сусідів * 20).

<pre>
ask turtles [
    let ratio-similar 0
    if total-nearby > 0 [
      set ratio-similar (similar-nearby / total-nearby)
    ]
    
    let disappearance-chance (different-nearby * 4) - (ratio-similar * 20)
    ...
</pre>

Додатково, якщо черепаха перед розрахунком не була щаслива, додаються ще 15% шансу зникнення

<pre>
if not happy? [
      set disappearance-chance disappearance-chance + 15
    ]
</pre>

Оскільки це шанс, він перевіряється випадково, і якщо випадковий процент вищий, **черепаха зникає**

<pre>
	if random 100 < disappearance-chance [
      die
    ]
</pre>

**Додано вивід інформації щодо кількості черепах** у двох видах

1. У вигляді графу:
	[Іллюстрація створеного графа](graph-illust.png)
	<pre>
		set-current-plot "Populations"
	</pre>
2. У вигляді монітора:
	[Іллюстрація створеного монітора](monitor-illust.png)
	<pre>
		globals[
			...
			number-turtles
		]
	</pre>

	<pre>
		to update-globals
		...
		  set number-turtles count turtles
	</pre>

<br>

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Модифіковано алгоритм пошуку клітини**: Спочатку черепаха перевіряє найближчі пусті клітини у радіусі 10 та обирає найбільш підходящу з них, якщо така є:
<pre>
	let my-color color
  
  let candidate-patches (patches in-radius 10) with [not any? turtles-here]
  
  if any? candidate-patches [
    let best-patch max-one-of candidate-patches [
      count (turtles-on neighbors) with [color = my-color]
    ]
    
    if best-patch != nobody [
      move-to best-patch
    ]
  ]
</pre>

Якщо ж такого патчу знайдено не було, то виконується стара логіка пошуку випадкової вільної клітини у радіусі 5, допоки така не буде знайдена

<pre>
	if not any? candidate-patches [
    rt random-float 360
    fd random-float 5
    setxy pxcor pycor
  ]
</pre>

**Додано логіку присвоєння певної форми кожному кольору черепахи**, причому одному явному кольору відповідає одна модель з пулу. Індивідуальність забезпечується попереднім перемішуванням списку форм

<pre>
	to setup
  clear-all

  set colors [ red green yellow blue orange ]
  set shapelist [ "butterfly" "bug" "box" "cow" "fish" "person" "plant" ]

  let chosenForms sublist (shuffle shapelist) 0 (length colors)
  
  ask n-of number patches [
    sprout 1
  ]
  ask turtles [
    let chosenIndex random number-of-ethnicities
    set color (item chosenIndex colors)
    set shape (item chosenIndex chosenForms)
  ]
</pre>

![Скріншот моделі в процесі симуляції](model-working.png)

Фінальний код моделі та її інтерфейс доступні за [посиланням](segregation_extended.nlogox).
<br>

## Обчислювальні експерименти
*// тут повинен бути наведений опис одного експерименту, за аналогією з першої л/р.* 
### 1. Вплив кількості націй черепах на фінальну їх кількість у моделі та час симуляції. 
Результат є середнім значенням з 5 проходів. Першопочаткова кількість черепах - 2500, граничний поріг сусідів - 100%.

<table>
	<thead>
		<th>Кількість націй</th>
		<th>Кількість черепах, що лишилися</th>
		<th>Модельний час</th>
	</thead>
	<tbody>
		<tr>
		<td>2</td>
		<td>16</td>
		<td>1829</td>
		</tr>
		<tr>
		<td>3</td>
		<td>32</td>
		<td>1556.8</td>
		</tr>
		<tr>
		<td>4</td>
		<td>63.2</td>
		<td>1383.4</td>
		</tr>
		<tr>
		<td>5</td>
		<td>77.2</td>
		<td>1302.2</td>
		</tr>
	</tbody>
</table>

[Графік залежності виживших черепах від кількості націй](graph-1.png)
[Графік залежності модельного часу від кількості націй](graph-2.png)

Як видно, зі зростанням кількості націй збільшується середній час симуляції та знижується кількість в остатку живих черепах, але з кожним новим проходом швидкість зростання часу і спадання кількості знижується, через те що початкове дроблення черепах на групи створює меншу їх кількість на групу при однаковій першопочатковій кількості. Це дає більшу можливість черепахам знайти підходящий патч, але і збільшує час переміщення або шанс зникнення у випадку, якщо такий патч не був знайдений.