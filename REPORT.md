# Отчет по лабораторной работе "Генерация последовательностей"

### Алексюнина Юлия, группа М8О-307
Номер в группе: 1, Вариант: 1 ((остаток от деления (1-1) на 6)+1)

### Цель работы

В данной лабораторной работе мне предстоит научиться генерировать последовательности с помощью рекуррентных нейронных сетей. В качестве последовательностей в зависимости выступает проза на русском языке, элемент последовательности - один символ.

### Используемые входные данные

Источник данных - тексты на http://lib.ru.

В качестве входных данных мною были выбраны следующие произведения:
1. Антуан де Сент-Экзюпери "Маленький Принц"

### Предварительная обработка входных данных

Считанный текст разделим на части по заданному количеству символов (maxlen) с фиксированным шагом (step). Чем больше maxlen, тем более "глубокими" становятся зависимости последующих символов от предыдущих. Также, чем меньше step, тем больше различных комбинаций символов попадет в обучающие данные.
```
maxlen = 40
step = 3
sentences = []
next_chars = []
for i in range(0, len(text) - maxlen, step):
    sentences.append(text[i: i + maxlen])
    next_chars.append(text[i + maxlen])
print('nb sequences:', len(sentences))
```
Преобразуем полученные последовательности символов в последовательности векторов, в которых каждому символу будет соотвествовать вектор [x1, x2, ..., xK, ... xN], где xK = 1, если K равно индексу данного символа, N = количество в нашем наборе chars, а все остальные значения равны нулю.
```
x = np.zeros((len(sentences), maxlen, len(chars)), dtype=np.bool)
y = np.zeros((len(sentences), len(chars)), dtype=np.bool)
for i, sentence in enumerate(sentences):
    for t, char in enumerate(sentence):
        x[i, t, char_indices[char]] = 1
    y[i, char_indices[next_chars[i]]] = 1
```
### Эксперимент 1: RNN

#### Архитектура сети

Приведу архитектуру сети RNN в виде последовательности слоёв. Определяем один скрытый слой LSTM с option['size'] = 256 единицами памяти. Для регуляции сети будем использовать слой Dropout с вероятностью 20, как правило, это делает узлы более устойчивыми к входам. Выходной уровень - это полносвязный (Dense) уровень, использующий функцию активации softmax для вывода прогнозирования вероятности для каждого из 51 символов в диапазоне от 0 до 1.
```
model1 = Sequential()
model1.add(SimpleRNN(option['size'], input_shape = (maxlen, vocab_size)))
model1.add(Dense(vocab_size)) 
model1.add(Activation('softmax'))
model1.summary()

Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
simple_rnn_1 (SimpleRNN)     (None, 128)               24064     
_________________________________________________________________
dense_1 (Dense)              (None, 59)                7611      
_________________________________________________________________
activation_1 (Activation)    (None, 59)                0         
=================================================================
Total params: 31,675
Trainable params: 31,675
Non-trainable params: 0
_________________________________________________________________
```
#### Результат

Обучение и график потерь во время обучения нейросети для каждого случая находятся в файле исходника

Иллюстрация к проекту
```
test_acc = model1.evaluate(x, y, verbose = 2)
print("Потери на данных")
print("%s: %.3f" % (model1.metrics_names[0], test_acc)) # loss (потери)
Потери на данных (по уже обученной нейросети)

loss: 1.051
```
Получившийся результат генерации:
```
ни никак не могут представить себе этот 
потровую шилобыл польвотав поремут.
 - ду, кой расть зреглы и когданих: гаровзо! в донь я будре трудо - вузныхраташнов еспь ина долая ни потут баласукий чество. но вытил савые чудо продол вылом ем. гоговели шик с чного. и учты ". мебарить пон мог, - с азакая нем радеть и потсли ним шедить десь озжезнь.
 ногов е прадужетал онобо вуркеши волочео - оприсмалоть б-ладата ствеспо.
 - покая, - какотвают у пнох..."и ризупель дебя да пестыте тетеко.
 - аси стался, чтостувенься, - спросил мол цветал сакдел нугды.
 - вотил мне редь? когда дежевдя гдямери"ь пробочитать, чтобы придемиять. планетеют, что да, он и инесилня:
 - коз чно бельнови воврим е пудеметел серицасям мне оме челнот сямнца. млочен ещи дов дали он бедь, ма, но я быв посомать и тарослюни гдома ся нас оде жизал онае склотый пусьнем никой и скараастллянни
 - значше моляшь... не пужемнять полихним я но не кардатселы тиросниросвар, что сприсли, ис м очуго увидут потолпат, провотруя в почши, л е пенахздулосе винцо.
 - кугода те твсемник, поковуг задьси гаядой мыл тыветыл смбетрезньядь, каск стень салься?
 - ворисны уременя о споинеть? то был не тепящикого покном, го олконужмыень огливая моесто, и тыл калы проспоть лесте саром чешь?


 xiiiiiъ


 - микопот, на огом. я весьволком.
 точерою пиинить, чтобу цветвид не вебеонь, и ответол жилсть о ток м даше сластееньй. и мылня.
 - он, жна носо мегланек... и подну спрося не ках вскола оночьоб в ссещеи тно молчиту шкал ина совсегдв и скуюдл гу.
ы- илисти мнечистоль онпещнижа те бебло
```
#### Вывод по данному эксперименту

Потери в сети уменьшались почти каждую эпоху, поэтому, я решила, что сеть сможет показать себя в лучшем свете при увеличении эпох, поэтому поставила значение отвечающее за их количество = 120, но, начиная с 114 эпохи, значения начали переставать улучшаться и извлечь какую-либо выгоду из обучения уже не оставалось возможным. Можно отметить некоторые замечания по поводу сгенерированного текста с использованием загруженной модели RNN. Заметны лишь несколько настоящих русских слов, весь текст лишен всяческого смысла. Но в некоторых местах можно увидеть попытки правильного соблюдения пунктуации.
Из-за того, что сам по себе русский язык сложен и многообразен (наличие падежей и родов, от которых зависят окончания слов), то RNN дает отнюдь не самый хороший результат. Возможно обучение сети на произведении, написанном на английском языке дало бы лучший результат.


Эксперимент 2: Однослойный LSTM
Архитектура сети
Приведу архитектуру однослойной LSTM в виде последовательности слоёв. Определяем один скрытый слой LSTM с option['size'] = 128 единицами памяти. Для регуляции сети будем использовать слой Dropout с вероятностью 20, как правило, это делает узлы более устойчивыми к входам. Выходной уровень - это полносвязный (Dense) уровень, использующий функцию активации softmax для вывода прогнозирования вероятности для каждого из 51 символов в диапазоне от 0 до 1.
```
    model2 = Sequential()
    model2.add(LSTM(option['size'], input_shape = (maxlen, vocab_size)))
    model2.add(Dropout(0.2))
    model2.add(Dense(vocab_size))
    model2.add(Activation('softmax'))
model2.summary()

Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
lstm_1 (LSTM)                (None, 128)               96256     
_________________________________________________________________
dropout_1 (Dropout)          (None, 128)               0         
_________________________________________________________________
dense_1 (Dense)              (None, 59)                7611      
_________________________________________________________________
activation_1 (Activation)    (None, 59)                0         
=================================================================
Total params: 103,867
Trainable params: 103,867
Non-trainable params: 0
_________________________________________________________________
```
### Результат
```
test_acc = model2.evaluate(x, y, verbose = 2)
print("Потери на данных")
print("%s: %.3f" % (model2.metrics_names[0], test_acc)) # loss (потери)
Потери на данных (по уже обученной нейросети)

loss: 0.505
```
Получившийся результат генерации:
```
земле. при свете луны я смотрел на его 
оданаков распотаться, торово. мен жичуго нечам, что она она все даваю себя, и вы слешим, что калечец.
 - ис1- спросте!
 - во - сказал маень, - поромол был какочик, заазам маленький принц рупываю, и ублисит с бском ме бые, чествее, эти отповаю, и свотачал какаю либу усталиник полэтол тот гобя, и дажи ущакол ни уго лишел посоветиразно: чно о виже у пялас. у потуспе. вся подилать ях..- скажал она щилто, как пробторей сереслия, смоллящих. - что это ка зезным нечус, ты уткажелся.
 - привум лисчет бымо дебя, кот полушь, де в ризоват водрох, ески я послы но мня та заход у сем сахдется, залочая мнельков. но чтобы ясле либы пихвету емлы будет заствалсть?
 - не, -ткого накогда ну мужу.
 - в дохжелся у мая заков,ть глафма..
 - пришимал?
 - нечествен..
 - да деже", - одномажил маленький принц был вед, пкищем, поваление солочао. это свошил ю слобия пранида малеський принц. - сказал он корочил нечим.
 - что было у какого линиве тводал. они даже ни говорму, - возпринллся был том покланеть загодансаказале:
 - точлочоон.
 - остобно, порослион дву маютнубу на вка прелешил...
 он наки полочий прощечего на олобы проста онеть делна. по-омо, как пробедть емо прислуть сказал себя, и они задеть. вак он иись люби, плинет смему в овиоправать зарреда.
 - ян. оннажин: в будем я ущал чело кказальесь пронечно, на он будешно. на зебуднои все все спразулае:
 - сказали...
 - это проко де хоел, он потему, онил покомол как я как да тих мне поковогваюл ест, что она у все просто мане киз. вое ходя оскалатья..
```
Вывод по данному эксперименту
Приближаясь уже к 100 эпохе, потери перестали уменьшаться на значимое количество единиц, но все же, по сравнению, с сетью RNN, после 120 эпохи дало результат = 0.9410, что меньше результата потерь у RNN на 120 эпохе ( = 1.6220) на 0.681. Изучая сгенерированный текст, я заметила, что символы разделены на словесные группы и стало намного больше настоящих русский слов. Некоторые слова в последовательности имеют смысл. В целом, текст смотрится более гармонично, чем предыдущий. У меня создалось впечатление, что я читаю какой-то старославянский текст.

## Эксперимент 3: Двухслойный LSTM
### Архитектура сети
Приведем архитектуру двухслойной LSTM сети в виде последовательности слоёв. Определим количество узлов для слоя LSTM, равное 256, и добавим еще второй слой LSTM. Теперь мы имеем уже два слоя LSTM, регулязацию которых выполняем при помощи техники dropout. Добавляем полносвязный слой с функцией активации - непрерывная функция softmax с 51 выходами, равное количеству уникальных символов в нашем тексте.
```
   model3 = Sequential()
   model3.add(LSTM(option['size'], input_shape = (maxlen, vocab_size), return_sequences = True))
   model3.add(Dropout(0.2))
   model3.add(LSTM(option['size']))
   model3.add(Dropout(0.2))
   model3.add(Dense(vocab_size))
   model3.add(Activation('softmax'))
model3.summary()

Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
lstm_1 (LSTM)                (None, 40, 256)           323584    
_________________________________________________________________
dropout_1 (Dropout)          (None, 40, 256)           0         
_________________________________________________________________
lstm_2 (LSTM)                (None, 256)               525312    
_________________________________________________________________
dropout_2 (Dropout)          (None, 256)               0         
_________________________________________________________________
dense_1 (Dense)              (None, 59)                15163     
_________________________________________________________________
activation_1 (Activation)    (None, 59)                0         
=================================================================
Total params: 864,059
Trainable params: 864,059
Non-trainable params: 0
_________________________________________________________________
```
### Результат
```
test_acc = model3.evaluate(x, y, verbose = 2)
print("Потери на данных")
print("%s: %.3f" % (model3.metrics_names[0], test_acc)) # loss (потери)
Потери на данных (по уже обученной нейросети)

loss: 0.781
```
Получившийся результат генерации:
```
еще отдавалось пенье скрипучего ворота,
 и вапил такум глотаки. и - понимался маленькимо принц.
 вот мни былу такогда так бы не кутку ну, она отбы радке то фвердцанед тводи никогта от, разны такее планетвыя. - это на будет я тучешь маленький принц. - на ответилия, ничебоя у немалобе
 - а ису уже поволясявся кого он истил...
 и после моголко, сладалка ответние", и месте не пуслы мотроко этом пришесля будат сомжать имо, коточьо на могже только поверяет: сы простомут, од наможени и росит, - сказал я посям шисмо бы что можпоима
 и еде не ствем, так межный прислы я цветко естил был так на издание, человык сказал най отвыту. что знах вилоскаваять умеетерьенко пробутал ето стали гостах: ты вынянал езразты росость!
 и он потночал сторжевнем он скуза.
 мветь, сказал маленький принц весли на симые! сабовится нассказы: 1вадые все пускуху. он был, что больско могрыхо, а ты топьенно, это сыло желпую. и потубал все ухоме отимал когда..
 xii

 скажа на дешь, я точешколсто постартяя больше взролкот.
 - просто слошил бороком.
 - этот это жи говорим...
 - подумал маленький принц спросли не сертею этот росто так мне самого био посекравума твепе.
.
 и ну хона хочет серед это же самореной полртому мог а как соброленнол у кая взрослые и резовитал може заводно я них он славал. твере, себя пустаня. тычиго бразустаяь тысчут лаз.. ты хотел, чтобы всо этой рию есень. и опланить ворох... схрисил маленькой денц.
 - все плакону поейвесте: я вись лы но чествеютка.
 - я все с насе веде иля мне ни остав тереейсе, вто ты исмиленься?
 - ничеободы 
```
### Вывод по данному эксперименту
По сравнению с полученными результатами у предыдущей сети, очевидно, что улучшилось качество сгенерированного текста. Время обучения также занимает большее количество времени нежели обучение однослойной нейросети LSTM, но при этом значение потерь улучшается быстрее, поэтому необходимость в количестве эпох = 120 отпадает, теперь я использую 30 эпох. Значение потерь равно = 0.781. Судя по полученному сгенерированному тексту, он улучшился, орфографических ошибок меньше, количество осмысленных русских слов тоже увеличилось, и сам текст выглядит более реалистичным, но все же совершенно бессмысленным. Если бы элементом последовательности выступало слово, а не как в моем случае - символ, мне кажется, что результаты были бы лучше.

## Эксперимент 4: Однослойный GRU
### Архитектура сети
Архитектура ондослойной GRU сети: количество узлов равно 256 для слоя GRU. Добавим слой Flatten для изменения формы тензора. Во избежание переобучения добавим Dropout. Затем добавляем полносвязный слой с функцией активации - непрерывная функция softmax с 51 выходами, равное количеству уникальных символов в нашем тексте.
```
model4 = Sequential()
model4.add(GRU(option['size'], input_shape = (maxlen, vocab_size), return_sequences = True))
model4.add(Flatten())
model4.add(Dropout(0.2))
model4.add(Dense(vocab_size)) 
model4.add(Activation('softmax'))
Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
gru_1 (GRU)                  (None, 40, 256)           242688    
_________________________________________________________________
flatten_1 (Flatten)          (None, 10240)             0         
_________________________________________________________________
dropout_1 (Dropout)          (None, 10240)             0         
_________________________________________________________________
dense_1 (Dense)              (None, 59)                604219    
_________________________________________________________________
activation_1 (Activation)    (None, 59)                0         
=================================================================
Total params: 846,907
Trainable params: 846,907
Non-trainable params: 0
_________________________________________________________________
```
### Результат
```
test_acc = model4.evaluate(x, y, verbose = 2)
print("Потери на данных")
print("%s: %.3f" % (model4.metrics_names[0], test_acc)) # loss (потери)
Потери на данных (по уже обученной нейросети)

loss: 0.201
```
Получившийся результат генерации:
```
иказания.
 "если я повелю своему генерал
 не ий потит я всенеедот митвое на и всром опил котолый пирь удит, - сказал маленький принц. - вод ребовет стул не спак..и посдизесли но интю еть. - вод овдлевеел да по плунитить но и знаел и оних путьтох, иконочен- зрпртошо верив. и загда,.... подаялси с оснему ниго брабетол у помол опкак.
 - значто, можни ды в нетемо озвавдетсл доманьа свераток п непет. маленький принц ниветве но од болот. тобы роб саревурисание вринама?
 - полаветь ветира. аве выем ез олобу нато, како пляленем пидотвнал всего веле быля кразаслов.и он два маленький принц. но все вотоби поденаедан, неськитоть од но измил...
 - и заход шеле. ни геду ноя, что ду ленние мволичать присловмо словно мелу в рускумелия, - скажилить ни он малыно.
 - асть нахордртвстко тврем я на ого долил не верковит бодот привеним споон приветитам не такее претеднел из на он. все пленелого вок расниу преволитые, что всем расмеет я  так если ов встришамило ня наченсенилого, потся дать смалечто. подрес ветом топощоелида, взвовму тапь,дчешя не илпооне, но дал
 маленьким принц. я мил он сприселит, - оконир с одмаретьки подровци и посяжнио свовитьрино вееди. нау меня превудела мни боться и. задовнивай сонему правелие на пыстепмноесят на мил порешика паственал, что маленький плинц.
, вае, в петер, у маленикой. ался и твезровут...
 -  на терекне.
 - все мне был на подправслис, в то был ше, - ствмне присляв.
 - понизь, то,  звокмительсо продим приня.
 - они, зканет и скожда табемния полнолить ен пикуважи водробу.
 - прастулот я ни толеное.
```
### Вывод по данному эксперименту
Полученные результаты мне нравятся больше всего, так как значение потери равно 0.201 - это лучший полученный результат. Мне показалось, что пунктуация в тексте приобрела новый уровень. Количество русских слов увеличилось, но качество самого текста все же не на высоте, он все еще напоминает бессмысленный старославянский, но это связано с посимвольным предсказанием, если бы использовались целые слова, текст содержал бы в себе больше ценности.

## Выводы
Из преимуществ RNN хотелось бы отметить, что она хорошо подходит для построения языковых моделей как посимвольных так и пословных. Главный плюс RNN в том, что она способна эффективно использовать данные с предыдущих шагов. RNN принимает на вход последовательность входных данных (в нашем случае символов). А, например, сверточной сети (CNN), на вход подается целое изображение. Для RNN же входными данными может служить как короткое или длинное предложение, так и целое произведение из тысячи абзацев. Это и является самым явным отличием RNN от традиционной нейронной сети. Более того, порядок, в котором подаются данные, может влиять на то, как в процессе обучения меняются матрицы весов и векторы скрытых состояний. К концу обучения в векторах скрытых состояний должна накопиться информация из прошлых шагов.

Однако у этих рекуррентных нейронных сетей есть и недостаток. В каждый момент времени они должны хранить информацию о входных данных за многочисленные предыдущие интервалы времени, но на практике такие протяженные зависимости не поддаются обучению. Это связано с проблемой затухания градиента, напоминающего эффект, наблюдающийся в сетях прямого распространения с большим количеством слоев: по мере увеличения количества слоев сеть в конечном итоге становится необучаемой.

Для решения этой проблемы был предложен слой долгой краткосрочной памяти (Long short-term memory, LSTM) который позволяет снова задействовать в процессе обучения предыдущую информацию и тем самым решить проблему затухания градиента. Слой управляемых рекуррентных блоков (Gated Recurrent Unit, GRU) основан на том же принципе, что и слой LSTM, однако рекуррентные блоки представляют собой более простые структуры по сравнению с LSTM и, соответственно, менее затратные в вычислительном смысле. В традиционных рекуррентных нейронных сетях для реализации обратной связи используется комбинация скрытого состояния на предыдущем шаге и текущих входных данных в слое с нелинейной функцией активации. В LSTM-сети обратная связь реализуется аналогично, но нейронных слоев используется не один, а четыре.

GRU – это вариант LSTM слоя, также обладающий устойчивостью к проблеме затухания градиента, но структура его проще, а потому и обучается он быстрее. Вместо трех вентилей в ячейки LSTM – входного забывания и выходного, в ячейке GRU всего два вентиля: обновления и сброса. Вентиль обновления определяет, какую часть предыдущего запомненного значения сохранять, а вентиль сброса – как смешивать новый вход с предыдущей памятью.

Оба эти метода разработаны для того, чтобы сохранять отдаленные зависимости в последовательностях слов (символов). Под отдаленными зависимостями имеются в виду такие ситуации, когда два слова или фразы могут встретиться на разных временных шагах, но отношения между ними важны для достижения конечной цели. LSTM и GRU отслеживают эти отношения с помощью фильтров, которые могут сохранять или сбрасывать информацию из обрабатываемой последовательности. Различие между двумя методами состоит в количестве фильтров (GRU – 2, LSTM – 3). Это влияет на количество нелинейностей, которое приходит от входных данных и в конечном итоге влияет на процесс вычислений.

Исходя из моего опыта, GRU обучаются быстрее и работают лучше, чем LSTM. Время, потраченное на одну эпоху у GRU в среднем равно 66.68 секунд, двухслойной LSTM - 154.1 секунд. Теоретически LSTM должны запоминать более длинные последовательности, чем GRU, и превосходить их в задачах, требующих моделирования отношений на расстоянии (long-distance relations).

Также, LSTM и GRU действительно показали лучшие результаты, чем простая RNN. Изучив текст, который сгенерировала сеть RNN, и тексты сетей с LSTM- и GRU-слоями, то у последних они были лучше. Очевидно, что первая не справляется с долгосрочными зависимостями в тексте, а, как и предполагалось, последним удается их обработать.

Таким образом, можно сделать вывод, что выбор архитектуры нейронной сети тесно связан со спецификой решаемой задачи. Решая задачу, стоит перебрать разные подходы и гиперпараметры и выбрать среди них те, которые смогут уловить закономерности в данном наборе текстов. Однако, в целом рекуррентные нейронные сети показывают гораздо более высокие результаты, чем простые нейронные сети или другие классификаторы.
