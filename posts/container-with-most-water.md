Container With Most Water
=========================

Oct 26, 2020 · 3 min read

На вход даётся `n` чисел представляющих собой высоты столбиков, как показано на рисунке:

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

Представим, что мы можем заполнить всё водой. В каком контейнере воды больше?

[Задача на LeetCode](https://leetcode.com/problems/container-with-most-water/).

Решение [#](#решение)
---------------------

По сути, нужно найти площадь прямоугольника. Определимся со сторонами:

![](/images/container-with-most-water--square.jpg)

Высота наименьшего столбика одна сторона (вода не должна переливаться через край), а другая сторона — расстояние между столбиками.

Казалось бы, можно взять два самых высоких столбика, это даст максимальную высоту. Однако, они могут находиться слишком близко друг к другу. Так что выгоднее может быть взять столбики поменьше, но зато подальше друг от друга.

Лучше написать брутфорс, чем не написать ничего, поэтому так и поступим. Всегда можно перебрать все возможные варианты за квадрат и найти максимум.

    /**
     * @param {number[]} height
     * @return {number}
     */
    var maxArea = function(height) {
      let result = 0;
    
      for (let i = 0; i < height.length; i++) {
        for (let j = i + 1; j < height.length; j++) {
          result = Math.max(result,
            Math.min(height[i], height[j]) * (j - i)
          );
        }
      }
      return result;
    };
    

![](/images/container-with-most-water--n2.jpg)

Такой себе рантайм. Можно лучше?

> Если интервьюер спрашивает «можно лучше» — ответ всегда да :-)

Попробуем определить где совершается лишняя работа на примере с картинки. Соберём все площади в массив, отсортируем и распечатаем.

![](/images/container-with-most-water--sort.jpg)

Откуда столько повторяющихся значений? Дело в том, что площадь определяется по _меньшей высоте_.

![](/images/container-with-most-water--ex.jpg)

В примере выше расстояние до `i` и `j` одинаковое, поэтому не имеет значение какой высоты столбики `i` и `j` если они выше. Более того, если один из них будет ниже, то мы гарантированно не получим площадь больше, площадь может только уменьшиться.

Это ключевой момент в переходе к линейному решению.

Посмотрим на перебор иначе. Будем начинать сравнение не с соседних элементов, а напротив — с краёв, т.е. фиксируем два максимально удалённых друг от друга столбика. Зачем? Чтобы одна из сторон прямоугольника гарантированно уменьшалась и не влияла на принятие решения о том куда двигаться дальше.

![](/images/container-with-most-water--n.jpg)

Далее, мы можем двигать один из двух столбиков: красный или синий, перебирая все площади. Однако, т.к. вторая сторона прямоугольника определяется по _минимальной высоте_, не может получиться так, что мы двигали красный столбик и получили где-то максимальную площадь.

Если встретили где-то столбики больше — это не важно, потому что минимальным всё равно остаётся синий. Если где-то меньше чем синий, тем более — площадь может только уменьшиться (помним, что второй параметр убывает и не может увеличить площадь).

Таким образом, вообще не имеет смысла сравнивать всех со всеми. На каждом шаге мы точно знаем где должен быть максимум отсекая `n` сравнений. Таким образом `n^2` превращается просто в `n`.

В этом и есть суть «жадинки»: на каждой развилке ты точно понимаешь какой путь приведёт к оптимальному результату, а какой точно нет, соответственно отметаешь какое-то количество заведомо неудачных вариантов для проверки.

    /**
     * @param {number[]} height
     * @return {number}
     */
    var maxArea = function(height) {
      let result = 0;
    +  let i = 0;
    +  let j = height.length - 1;
    
    -  for (let i = 0; i < height.length; i++) {
    -    for (let j = i + 1; j < height.length; j++) {
    +  while (i < j) {
        result = Math.max(result,
          Math.min(height[i], height[j]) * (j - i)
        );
    +    // всегда двигаем указатель на меньший столбик,
    +    // т.к. заранее знаем, что, двигая указатель на больший,
    +    // большей площади там не будет
    +    if (height[i] < height[j]) {
    +      i++;
    +    } else {
    +      j--;
    +    }
    -   }
      }
      return result;
    };
    

![](/images/container-with-most-water--n-result.jpg)

Так намного лучше! Ускорение на порядок.

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.