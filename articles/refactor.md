
Среди начинающих разработчиков постоянно вспыхивают споры что лучше технология X или Y. Среди современных php фремворков обычно сравнивают yii и laravel. Как мне кажется, вопрос не корректен, ведь у каждого человека свои пристрастия и каждому ближе какое-то свое решение.

Некоторое время назад мне пришлось проводить рефакторинг существующего кода. Код отвечал за работу апи сервера на бэкэнде. Код был написан в процедурном стиле и представлял из себя идин большой switch-case. Обращение к базе банных тоже были в процедурном стиле. Все настройки были вынесены в глобальные переменные.

Для уменьшения сложности работы в первую очередь я изменил работу с БД. Я переписал все обращения к БД на pdo. Внедряя pdo, я выпустил на свет все проблемы, ошибки и неточности, которые до этого игнорировались, и сделал код более стабильным.

Внедрение pdo увеличило количество глобальных переменных. Для того чтобы разобраться с глобальными переменными, я поставил [pimple](http://pimple.sensiolabs.org/) - и перенес в него все глобальные переменные и настройки pdo. Теперь вместо 15 глобальных переменных стала одна.

После этого я решил уменьшить размер файла. Я разбил файл на два - в одном оставил свитч кейс. В другой перенес методы которые вызывались из этого свича. Это позволило сделать немного более приятной работу как с первым файлом так и со вторым + это позволило немного уменьшить дублироваие код, за счет его переиспользования.
Следующим этапом методы стали разноситься в разные специализированные класссы и групироваться вокруг таблиц, которые они используются. Параллельно с этим стали появляться библиотеки и сторонние модули в которые собирался код используемый разными модулями. 
Где то в этот момент я осознал что я пишу свой собственный маленький фреймворк. В нем уже оформился контейнер для храненния всего, система роутинга, начали оформляться компоненты и система контроллеров. При этом каждый файл был полон инклудов. 
Так как инклуды стали порядочно мешать я решил использовать автозагрузчик композера. Он быстро и дешево навел порядок и оставил только один вызов самого себя в индексе.
Через какое то время мне понадобилось описать достаточно сложную логику работы с записями таблицы. Так как я уже понимал что я создаю свой фреймворк - я подключил улоквиент и создал модель на его основе. 
Таким образом мой фреймворк окончательно оформился.

Если подумать, то я прошел путь от зрз4 до пхп5 только что. Скорее всего использование уии дало бы более быстрое приложение, но использование компонентной модели позволяет постепенно наращивать функционал.

Yii построен на принципе монолитности - один фрейворк предлагает в себе все необходимые решения, для того чтобы сделать минимальный сайт. Laravel построен по принципу сборной конструкции: каждый его компонент может жить в отдельности. Структура yii потенциально обеспечивает большую быстродействие, так как позволяет произвести оптимизацию кода и ресурсов. Структура ларавела дает больше гибкости.