Combination Sum
===============

Feb 1, 2021 · 4 min read

Найти все _уникальные_ комбинации указанных чисел, которые в сумме дадут заданное число. Одно и то же число может быть использовано _сколько угодно раз_.

Гарантируется, что в ответе _не будет_ более 150 комбинаций, а всего кандидатов не более 30. Все кандидаты — _положительные целые числа_, без повторений.

Пример.

*   Input: `candidates = [2,3,6,7], target = 7`
*   Output: `[[2,2,3],[7]]`

Из 2 и 3 получим 2 + 2 + 3 = 7. Обратите внимание, что 2 используется несколько раз. Из 7 можно получить только 7 = 7.

В итоге, две комбинации.

Решение [#](#решение)
---------------------

На самом деле, есть несколько вариантов этой задачи с различными ограничениями (на литкоде надо добавить ii, iii, iv, и т.д. в название задачи в URL).

Ограничения в этой задаче максимально всё упрощают, но тем лучше, чтобы посмотреть на бектрекинг. Итак, основная идея бектрекинга — проверить дерево возможных вариантов, и найти те, которые подходят под условия задачи.

Гарантируется, что в ответе не будет более 150 комбинаций, что довольно мало и хорошо подходит для рекурсии.

Каким образом строить это «дерево возможных вариантов»? Нам нужно проверить все возможные суммы, которые можно составить из указанных кандидатов.

В рекурсии важно подумать о двух вещах:

*   базовый случай, т.е. выход из рекурсии
*   рекурсивный переход

Поговорим про каждый пункт отдельно.

Если мы начинаем складывать кандидатов в некоторую сумму и она превышает указанный `target` — дальше суммировать смысла нет, потому что все кандидаты положительные, соответственно, сумма может только увеличиться. Это и есть условие для выхода.

Теперь про рекурсивный переход.

«Взять следующее число в сумму» означает вычесть его из `target` и вызвать ту же функцию с новым `target` (можно и с нуля вверх идти, особой разницы нет). На стеке нужно накапливать «текущий путь»: что за числа были использованы.

    // положим на стек кандидата
    st.push(candidate);
    // попробуем вызвать ту же функцию для нового target,
    // который меньше на указанного кандидата
    dfs(target - candidate, st);
    // как только вышли из рекурсии важно снять со стека слагаемое,
    // потому что мы уже его проверили
    st.pop();
    

Получается, обход дерева «возможных вариантов» в глубину. В кавычках — потому что на самом деле построенного дерева, в смысле структуры данных для обхода, нет, а сама рекурсия и формирует дерево.

Каким образом брать только уникальные комбинации? Конечно, можно завести сет, и складывать результаты туда, но это сериализация в строки… будет работать, но можно быстрее.

На примере `candidates = [2,3,6,7], target = 7`.

![](/images/combination-sum--tree.jpg) (картинка кликабельная)

Заходя в рекурсию, будем запоминать начало в списке кандидатов и начинать следующий ряд проверок с него, таким образом отсекая варианты с переменой мест слагаемых.

> Как всегда в задачах на рекурсию полезно нарисовать дерево возможных вариантов ручками, прежде чем писать код.

    /**
     * @param {number[]} candidates
     * @param {number} target
     * @return {number[][]}
     */
    var combinationSum = function(candidates, target) {
      const result = [];
    
      function dfs(start, target, st = []) {
        // базовый случай — дальше складывать смысла нет.
        // Все кандидаты положительные по условию,
        // а значит дальше target будет только уменьшаться
        if (target < 0) {
          return;
        }
        // это нужная нам кобинация,
        // положим в ответ
        if (target === 0) {
          // важно здесь скопировать массив!
          // иначе последующие изменения по ссылке
          // всё испортят
          result.push([...st]);
        } else {
          // начинаем следующий ряд проверок кандидатов
          // не с самого начала (ноль), а с некоторого start,
          // т.е. кандидата который был проверен ранее,
          // чтобы избежать комбинаций с перестановкой слагаемых
          for (let i = start; i < candidates.length; i++) {
            // кладем на стек текущего кандидата
            st.push(candidates[i]);
            // создаём «новый ряд дочерних узлов»
            // в дереве возможных вариантов
            dfs(i, target - candidates[i], st);
            // не забываем снять со стека
            // отработанный вариант
            st.pop();
          }
        }
      }
    
      // запускаем рекурсивных обход,
      // начиная с первого элемента в списке кандидатов
      // и изначальный target
      dfs(0, target);
    
      return result;
    };
    

Если аккуратно нарисовать дерево возможных вариантов, становится ясно, что можно ещё немного ускориться если отсортировать всех кандидатов. Как только target становится отрицательным, то в цикле можно сделать `break` — мы точно знаем, что все следующие рекурсивные вызовы упрутся в базовый случай, а значит их сразу можно пропустить.

    var combinationSum = function(candidates, target) {
      const result = [];
    
    + candidates.sort((a, b) => a - b);
    
      function dfs(start, target, st = []) {
        if (target < 0) {
          return;
        }
        if (target === 0) {
          result.push([...st]);
        } else {
          for (let i = start; i < candidates.length; i++) {
    +       if (target - candidates[i] < 0) {
    +         break;
    +       }
            dfs(i, target - candidates[i], st);
            st.pop();
          }
        }
      }
      dfs(0, target);
      return result;
    };
    

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.