# Форматирование html-разметки
Функция, форматирующая html-разметку по заданным правилам. Обладает следующими особенностями:
* малый вес (в сжатом виде 2.3 kB);
* отсутствие каких-либо зависимотсей;
* простая, и в то же время обладающая богатыми возможностями, конфигурация правил;
* лицензия [MIT](LICENSE).

##Применение:
```javascript
htmlFormatting(node, valid_elements);
```
* **node** - элемент DOM, дочерние элементы которого будут отформатированы в соответствии с правилами (сам же контейнер при этом затронут не будет);
* **valid_elements** - объект, содержащий правила форматирования.
 
##Кофигурирование:
**Примечание:** так как функция писалась в первую очередь для использования в сочетании с редактором TinyMCE, поля объекта, содержащего правила, именуются по приницпу *under_score*, в соответствии с правилами конфигурации самого редактора TinyMCE.

Каждое правило форматирования - это набор тегов, разделенных запятыми, которым соответствует объект с определенными установками. Объект установок правила может содержать следующие поля:

* **convert_to** - указание сконверитровать данный элемент в другой;
* **valid_styles** - перечень разрешенных стилей, разделенных запятыми;
* **valid_classes** - перечень разрешенных классов, разделенных пробелами;
* **no_empty** - указание удалять пустые элементы;
* **process** - функция, в которой вы можете определить какую-либо дополнительную обработку элемента;
* **valid_elements** - отдельный набор правил для дочерних элементов данного элемента.
 
К примеру, следующая конфигурация разрешит вставку только заголовков и параграфов, при этом у заголовков будут удалены все стили и классы:
```javascript
valid_elements = {
    'h1,h2,h3,h4,h5,h6': {
        valid_styles: '',
        valid_classes: ''
    },
    'p': {}
}
```

А вот конфигурация, которая применяется у нас в проекте (мы использум данную функцию для форматирования текста, вставляемого редакторами в TinyMCE из внешних источников):
```javascript
headerRule = {
    'br': {
        process: function (node) {
            var parent = node.parentNode,
                space = document.createTextNode(' ');

            parent.replaceChild(space, node);
        }
    }
},

valid_elements = {
    'h1': {
        convert_to: 'h2',
        valid_styles: '',
        valid_classes: '',
        no_empty: true,
        valid_elements: headerRule
    },
    'h2,h3,h4': {
        valid_styles: '',
        valid_classes: '',
        no_empty: true,
        valid_elements: headerRule
    },
    'p': {
        valid_styles: 'text-align',
        valid_classes: '',
        no_empty: true
    },
    a: {
        valid_styles: '',
        valid_classes: '',
        no_empty: true,

        process: function (node) {
            var host = 'http://' + window.location.host + '/';
            if (node.href.indexOf(host) !== 0) {
                node.target = '_blank';
            }
        }
    },
    'br': {
        valid_styles: '',
        valid_classes: ''
    },
    'blockquote,b,strong,i,em,s,strike,sub,sup,kbd,ul,ol,li,dl,dt,dd,time,address,thead,tbody,tfoot': {
        valid_styles: '',
        valid_classes: '',
        no_empty: true
    },
    'table,tr,th,td': {
        valid_styles: 'text-align,vertical-align',
        valid_classes: '',
        no_empty: true
    },
    'embed,iframe': {
        valid_classes: ''
    }
}
```

Мы запрещаем вставку изображений с внешних источников, к тому же у всех разрешенных элементов удаляем какие бы то ни было
классы. Кроме того, так как у нас на странице всегда присутствует заголовок первого уровня (название статьи), то при
копировании таких заголовков преобразовываем их в заголовки второго уровня. Так же, согласно нашим внутренним правилам, мы
удаляем все дочерние элементы из заголовков, а переносы (`<br>`) заменяем на пробелы. Из стилей мы сохраняем все стили для
встраиваемых элементов (`embed`, `iframe`), `text-align` для параграфов и таблиц, а так же `vertical-align` только для таблиц.
Ну и напоследок, к ссылкам на внешние ресурсы добавляется атрибут `target="_blank"`.

Независимо от правил конфигурации у всех элементов будут всегда удаляться идентификаторы, при их наличии, а в тексте будут заменяться неразрывные пробелы на обычные.
