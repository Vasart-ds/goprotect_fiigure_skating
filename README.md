# Описание проекта "GoProtect"
Доброго дня! 

В этом проекте мы попробуем создать модель, прогнозирующую вероятность успешного выполнения элемента/набора элементов. 

<details >
    <summary style="background-color: lightblue;"><b>ОПИСАНИЕ ДАННЫХ</b></summary>

<b>units</b>

<li>id: идентификатор юнита</li>
<li>color: категория</li>
<li>school\_id: идентификатор школы</li>

<b>tournaments</b>

Турнир состоит из нескольких категорий, оценки по категориям расписаны в total\_scores

<li>id: идентификатор турнира</li>
<li>date\_start: дата начала</li>
<li>date\_end: дата завершения</li>
<li>origin\_id: место проведения</li>

<b>total_scores</b>

Оценки за выступления по категориям и общие за турнир

<li>id: идентификатор выступления, джойнится с tournament\_scores.total\_score\_id</li>
<li>unit\_id: идентификатор юнита, ключ к units.id</li>
<li>tournament\_id: идентификатор турнира, tournaments.id</li>
<li>components\_score: артистизм (мастерство, композиция, хореография)</li>
<li>base\_score: базовая оценка за элементы в выступлении (идеал)</li>
<li>elements\_score: реальная оценка всех выполненных элементов, base\_score+goe</li>
<li>decreasings\_score: снижения оценок за ошибки</li>
<li>total\_score: components\_score+elements\_score+decreasings\_score за выступление</li>
<li>starting\_place:
<li>place: занятое место в категории category\_name+segment\_name</li>
<li>segment\_name: название сегмента</li>
<li>info: комментарии и пояснения к оценке</li>
<li>overall\_place: итоговое место в турнире</li>
<li>overall\_total\_score: итоговая оценка за весь турнир</li>
<li>overall\_place\_str: комментарии, пояснения</li>

<b>tournament\_scores</b>

Таблица с оценками поэлементно

<li>id: идентификатор оценки за конкретный элемент/комбинацию</li>
<li>total\_score\_id: идентикатор выступления, ключ total\_scores.id</li>
<li>title: запись элемента или комбинации элементов с отметками об ошибках</li>
<li>decrease: за что снижена оценка</li>
<li>base\_score: базовая оценка (идеал, цена данного элемента/комбинации, сложность)</li>
<li>goe: Grade of Execute, качество исполнения, судейские надбавки/убавки</li>
<li>avg\_score: оценка за элемент/комбинацию (усредненная по судьям)</li>

Расшифровка элементов tournaments\_scores.title

Разбираем только одиночное катание. Есть 3 типа элементов:

- Прыжки: начинаются с цифры от 1 до 4, потом идет код прыжка, потом может стоять один из специальных кодов
- Вращения. Сначала идет код вращения, после которого стоит уровень (B – базовый, 1, 2, 3,
  - Если после элемента стоит NV – not value значит элемент не выполнен.
- Шаги. Два варианта. Может быть так же как у вращений 5 уровней и NV – StSqB, StSq1 и тд.
- Внимание:
  - Могут попадаться базовые уровни без литеры В, например ChSt или CCoSp
  - Иногда прыжки иногда указаны без цифры
  - Также могут встречаться отдельные элементы из других групп
- Ошибки:
  - q – недокрут прыжка в четверть оборота. Базовая стоимость при такой ошибке остается неизменной, но судьи обязательно снизят за это GOE
  - < – спортсмен провращался в воздухе на 90°-180 ° меньше, чем положено, "недокрутил". Стоимость прыжка за такую ошибку не сильно, но снижают.
  - << – спортсмен недокрутил более 180°. Стоимость прыжка становится, как если бы прыгнул на один оборот меньше
  - e – Этот знак ставится, когда фигурист отталкивается от льда с неправильного ребра. Правильные ребра: на лутце – наружное, на флипе – внутреннее. На базовую стоимость влияет ровно на то же количество баллов, что и <
  - ! – Этот знак так же ставится только у флипа и лутца в случае, если технический специалист посчитал, что отрыв происходит с "нечеткого ребра". На базовую стоимость прыжка эта ошибка не влияет, но судьи обязательно её учтут при выставлении GOE
  - COMBO – Не выполнен обязательный каскад в короткой программе. Этот знак после прыжка, например: 3Lz+COMBO, говорит о том, что спортсмен должен был исполнить каскад прыжков, но, по какой-то причине (чаще всего падение), не смог. На оценку не влияет, но оставляет плохое впечатление у судейской бригады.
  - REP – Обозначает ошибку, похожую на COMBO, но в произвольной программе. По правилам, в произвольной программе фигурного катания один и тот же прыжок второй раз можно исполнить только в составе каскада или комбинации. Если по каким то причинам спортсмен оба раза прыгнул прыжок сольно, то ко второй попытке добавляют этот знак и базовую стоимость уменьшают на 30%.
  - SEQ – Комбинация прыжков. Фигурист сразу после любого прыжка делает аксель. В этом случае SEQ означает, что была исполнена комбинация прыжков. Раньше базовая стоимость прыжков, исполненных в комбинации, умножалась на коэффицент 0.8, с сезона 2022 стоимость комбинации приравнивается к каскаду
- Бонус:
  - B - ознает бонус за элемент.

○ x – Элемент исполнен во второй половине программы. Отностится только к прыжкам! Если прыжок сделан во второй половине программы, его базовая стоимость умножается на коэффициент 1.1. Недавно введено ограничение – только три последних прыжка получат бонус. Ввели это ограничение, потому что многие спортсмены переносили все свои прыжки во вторую часть.
    </details>
