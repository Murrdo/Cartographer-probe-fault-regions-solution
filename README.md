# Cartographer-probe-fault-regions-solution

Этот гайд может помочь если у вы столкнулись с тем, что картографер снимает неверный меш и у вас есть одна или несколько полос/областей более менее параллельных осям принтера с провисающим первым слоем или слоем "гармошкой", и при этом у вас нет реального перекрута осей.

Результат до:

![image](https://github.com/user-attachments/assets/9d96c158-fff3-453d-8174-94d98f0a32e6)


Результат после:

![image](https://github.com/user-attachments/assets/6abfd4bc-abc3-440b-85c4-f768bd32a737)


Возможно проблема вызвана неоднородностью "магнитных свойств" каждой конкретной пластины, у меня из 4х пластин в наличии на всех были проблемные области, при повороте пластины на 90гр область поворачивается вместе с пластиной, реального перекрута осей нет, на механическом таче результат хороший. Но при этом есть НЕБОЛЬШОЕ сходство проблемных областей на разных пластинах, так что может быть это результат взаимодействия пластины с "магнитным полем" всего принтера и/или стола и магнита.

Идея в том, что бы использовать компенсацию перекрута осей, при том что реального перекрута может и не быть. Но проблема в том, что компенсация перекрута даже близко не задумывалась использоваться таким образом как здесь, её задача устранить погрешность в десятку-две по всей длинне стола, а не бороться с локальным "горбом" длинной в десятки миллиметров.

Так или иначе - штатные способы калибровки перекрута не работают, что авто, что в ручную даже по 30 точкам - по какой то причине после неё всё равно остаются проблемные области, их расположение немного меняется, но могут появиться и новые полосы нависаний/гармощек, в общем результат после ручной штатной калибровки я не могу назвать даже минимально приемлимым за 3 попытки по 20-40 точкам.

### Подготовка:

Убрать (если есть) секцию AXIS_TWIST в самом низу конфига, иначе новая калибровка будет происходить **ПОВЕРХ** уже работающего AXIS_TWIST и вы запу таетесь в результатах. 
Перезагрузить прошивку

Откалибровать модель стола картографера в ручную - 
```
CARTOGRAPHER_CALIBRATE METHOD=MANUAL
```

Перезагрузить прошивку

В любом удобном месте конфига создаём  секцию

```
[axis_twist_compensation]
#для упролщения процесса советую создать здесь "линейку" что бы было проще ориентироваться к каким координатам примениться офсет.
# Но линейку нужно спозиционировать по несколькоим точкам, т.е. например меняется несколько значений в zy_compensations на +1, перезагружаетесь
#- снимаете меш, смотрите какая точка какой координате соответствует и заполняете линейку для что бы координаты соответствовали реальным.
#-----------        20     30     40     50      60    70     80     90     100   110    120     130    140   150     160    170    180    190   200    210    220    230    240    250    260    270    280    290    300    310    320     330
zy_compensations = +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00, +0.00,
#здесь мы создаёт массив точек (офсетов) для корректировки дефекта, их количество должно быть НЕЧЁТНЫМ,
#[количество]/[max-mix]=расстояние между точками, его следует выбирать соразмерным дефективной области и крутизне её начала или конца
compensation_start_y = 
compensation_end_y = 
#координаты начала области калибровки и её конца, лучше делать во весь стол, я пробовал задавать компенсацию только
#в проблемной области но работает криво. Основной момент - точки должны быть СИММЕТРИЧНЫ относительно [bed_mesh] zero_reference_position: 164, 162 
#для конкретной оси. Тут же мы указываем ось, вдоль какой оси дефектная область - ту ось и указываем
```


#### Калибровка:

Далее наша задача любым удобным образом составить представление о проблемной области:

1.Можно просто напечатать первый слой во весь стол, снять пластину и измерить линейкой примерные координаты её начала или конца
2.Можно заранее снять качественный детальный меш механическим тачем и сравнить его с тем что выдает картографер

Например, вот результат механической пробы:

![image](https://github.com/user-attachments/assets/039616d5-30b7-47cb-916e-068304cecd38)


А это меш снимаемый картографером

![image](https://github.com/user-attachments/assets/b5398d17-9799-4240-8ee5-c1dcf52f66d5)


Сопоставляя график и проблемные области на первом слое видимо две проблемные области, первая небольшая в диапазоне Y65-Y100, вторая огромная по  Y185-Н270.

И начинаем в ручную корректировать коэффициенты в секции перекрута пересканируя стол пока не получите график близкий к механике.

Запредельной точности не нужно, отклонение в 2-3 сотки  на качестве первого слоя не сказывается.

Очень важный момент - если точка хоуминга у вас внутри проблемной области - после каждого изменения возможно есть смысл сделать ручную калибровку модели картографера, 
```
CARTOGRAPHER_CALIBRATE METHOD=MANUAL
```
 и возможно даже 
 ```
CARTOGRAPHER_THRESHOLD_SCAN
```

По завершению/перед тестовой печатью обязательно перекалибровываем модель картографера 
```
CARTOGRAPHER_CALIBRATE METHOD=MANUAL
```

Финально корректируем офсеты по реальной печати первого слоя, можно уже не во весь стол главное бы тест пересекал проблемную область.

Результат после подбора значений, дающий хороший первый слой




![image](https://github.com/user-attachments/assets/f5d1a68d-106c-4e14-a717-958b1032b12c)


Из разряда неудобств - это придеться делать под каждую проблемную пластину, а при установке беспроблемной пластины секцию перекрута нужно удалять из конфига
