# Slice

**Slice** представляет собой обёртку над типом array. Тот же смысл, но не требует
ручного управления памятью.

Представляет собой структуру со ссылкой на массив с данными, длину и вместимость.
**Длина** – фактическое количество элементов в массиве. **Вместимость** – размер 
аллоцированного базового массива от первого элемента по ссылке.

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

Указатель в структуре ведёт на первый элемент слайса.

### Comparison


### Increasing underlying array

При превышении вместительности запускается подсчёт новой длины основного массива:
- Если добавлено количество элементов, которое превышает текущую вместительность
более чем в два раза, аллоцируется массив с длинной, равной количеству всех (старых
и новых) элементов.
- Если вместительность старого массива меньше 256, то он просто удваивается.
- В остальных случаях длина основного массива увеличивается с коэффициентом 1.25
циклом с прибавкой значения: `newcap += (newcap + 3*threshold) >> 2`.

```go
func nextslicecap(newLen, oldCap int) int {
	newcap := oldCap
	doublecap := newcap + newcap
	if newLen > doublecap {
		return newLen
	}

	const threshold = 256
	if oldCap < threshold {
		return doublecap
	}
	for {
		// Transition from growing 2x for small slices
		// to growing 1.25x for large slices. This formula
		// gives a smooth-ish transition between the two.
		newcap += (newcap + 3*threshold) >> 2

		// We need to check `newcap >= newLen` and whether `newcap` overflowed.
		// newLen is guaranteed to be larger than zero, hence
		// when newcap overflows then `uint(newcap) > uint(newLen)`.
		// This allows to check for both with the same comparison.
		if uint(newcap) >= uint(newLen) {
			break
		}
	}

	// Set newcap to the requested cap when
	// the newcap calculation overflowed.
	if newcap <= 0 {
		return newLen
	}
	return newcap
}
```

### Questions

> Вопрос: А зачем вообще нужен capacity?\
> Ответ: Чтобы понимать, когда произойдёт аллокация нового базового массива. Т.к. 
> массив – это неразрывная последовательность в памяти, при превышении вместимости 
> происходит аллокация нового базового массива и копирование данных из старого.

> Вопрос: Как работает make со слайсами?\
> Ответ: make создаёт новый слайс и аллоцирует базовый массив в зависимости от 
> переданных параметров. Стоит помнить, что аллоцированный массив состоит из 
> нулевых значений типа массива. Но, т.к. длина слайса может быть указана отдельно,
> мы можем получить пустой слайс.

> Вопрос: Что будет, если передать нулевой cap в make?\
> Ответ: 

### Links

- [Go Blog. Go Slices: usage and internals](https://go.dev/blog/slices-intro#TOC_5.)
- [Demystifying Golang Slices](https://medium.com/@andreiboar/demystifying-golang-slices-83ffe3550db5) 