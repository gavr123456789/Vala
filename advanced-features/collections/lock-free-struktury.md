# Lock-free структуры

**Неблокирующая синхронизация** — подход в [параллельном программировании](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D1%80%D0%B0%D0%BB%D0%BB%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5) на [симметрично-многопроцессорных системах](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%BC%D0%BC%D0%B5%D1%82%D1%80%D0%B8%D1%87%D0%BD%D0%B0%D1%8F_%D0%BC%D1%83%D0%BB%D1%8C%D1%82%D0%B8%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80%D0%BD%D0%BE%D1%81%D1%82%D1%8C), в котором принят отказ от традиционных примитивов [блокировки](https://ru.wikipedia.org/wiki/%D0%91%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29), таких, как [семафоры](https://ru.wikipedia.org/wiki/%D0%A1%D0%B5%D0%BC%D0%B0%D1%84%D0%BE%D1%80_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29), [мьютексы](https://ru.wikipedia.org/wiki/%D0%9C%D1%8C%D1%8E%D1%82%D0%B5%D0%BA%D1%81) и [события](https://ru.wikipedia.org/wiki/%D0%A1%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B5_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29). Разделение доступа между потоками идёт за счёт [атомарных операций](https://ru.wikipedia.org/wiki/%D0%90%D1%82%D0%BE%D0%BC%D0%B0%D1%80%D0%BD%D0%B0%D1%8F_%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F) и специальных, разработанных под конкретную задачу, механизмов блокировки.

Преимущество неблокирующих алгоритмов — в лучшей [масштабируемости](https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D1%81%D1%88%D1%82%D0%B0%D0%B1%D0%B8%D1%80%D1%83%D0%B5%D0%BC%D0%BE%D1%81%D1%82%D1%8C) по количеству процессоров. К тому же, если ОС прервёт один из потоков фоновой задачей, остальные, как минимум, выполнят свою работу, не простаивая. По максимуму — возьмут невыполненную работу на себя.

В Библиотеки Gee присутствуют неблокирующие структуры:

## [ConcurrentList](https://valadoc.org/gee-0.8/Gee.ConcurrentList.html) 

Односвязный список. Эта реализация основана на статье Михаила Фомичева и Эрика Рупперта.

{% embed url="http://www.cse.yorku.ca/~ruppert/papers/lfll.pdf" %}

## [ConcurrentSet](https://valadoc.org/gee-0.8/Gee.ConcurrentSet.html)

Список с пропусками \(Skip list\). Эта реализация основана на магистерской диссертации Михаила Фомичева.

{% embed url="http://www.cse.yorku.ca/~ruppert/Mikhail.pdf" %}



