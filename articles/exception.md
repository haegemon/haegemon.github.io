Исключения - это одна из тем, которые не пользуются популярностью в php. Возможно основная причина в том, что  без исключений можно обойтись. Однако исключения позволяют уменьшить количество условных операторов в вашей программе и, как следствие, уменьшить сложность самого кода. Кроме этого основная проблема с функциями, которые работают без исключений, это необходимость всегда держать в памяти и обрабатывать все состояния, которые эти функции могут вернуться. 

Сами по себе исключения не являются чем-то сложным. Они хорошо описаны в [различных статьях](http://habrahabr.ru/post/99431/). Если у вас есть код написанный без исключения, то всегда можно провести рефактиринг и привести код к хорошему состоянию. Неплохим примером будет небольшой рефакторинг который я недавно делал для [rainlab/blog](https://octobercms.com/plugin/rainlab-blog).


Как я уже [писал](http://pawelsamusev.com/post/octobercms) для своего блога я использую [rainlab/blog](https://octobercms.com/plugin/rainlab-blog). Я всегда просматриваю тикеты проекта, чтобы понимать, что от него можно ожидать. Увидев один тикет, я был очень смущен:
> Doesn't show 404 page when post is not exists

Я попробовал повторить это у себя в блоге и получил 500 ошибку:
[screenshot with 500 error]

Когда я заглянул в исходный код проекта, то увидел следующий код:

`
class Post extends ComponentBase {

    public function onRun()
    {
        $this->categoryPage = $this->page['categoryPage'] = $this->property('categoryPage');
        $this->post = $this->page['post'] = $this->loadPost();
    }
    protected function loadPost()
    {
        $slug = $this->property('slug');
        $post = BlogPost::isPublished()->where('slug', $slug)->first();
        /*
         * Add a "url" helper attribute for linking to each category
         */
        if ($post && $post->categories->count()) {
            $post->categories->each(function($category){
                $category->setUrl($this->categoryPage, $this->controller);
            });
        }
        return $post;
    }
}
`

Поясню: мы пытаемся найти страницу по ее адресу. Если страница есть, то объект `$post` найден и он передастся дальше. А если страница не найдена, то дальше передается пустой объект. Тут стоит обратить внимаение на условие. В нем проверяется есть ли объект `$post`. Но если его нет, то ничего не происходит. 

Теперь посмотрим какой код советовали написать разработчики на странице материала:

`
<?php
function onEnd()
{
        $this->page->title = $this->post->title;
}
?>
`

Тут становится понятно почему появилась 500 ошибка: материал не найден, получен пустой объект, но система пытается задать имя для страницы все равно.

Улучшенная версия этого же кода является следующей:
`
<?php
function onEnd()
{
    // Optional - set the page title to the post title
    if (isset($this->post))
        $this->page->title = $this->post->title;
}
?>
`
Стоит опять обратить внимание на условный оператор. Мы уже в двух местах видим проверку на существование объекта `$post`. Мы можем улучшить код этот код. Сразу после того, как мы нашли объект `$post`, мы должны проверить, что объект ну пустой. Если это не так, то прервать текущие действия и перейти на страницу 404 ошибки.

В laravel встроенны прекрасные методы работы с объектами, которые могут вызывать исключения если объект не найден. Используя их, можно переписать код нахождения страницы:

`

use App;
class Post extends ComponentBase
{
    public function onRun()
    {
        $this->categoryPage = $this->page['categoryPage'] = $this->property('categoryPage');
        try{
            $this->post = $this->page['post'] = $this->loadPost();
        } catch (ModelNotFoundException $e){
            return App::make('Cms\Classes\Controller')->setStatusCode(404)->run('/404');
        }
    }
    protected function loadPost()
    {
        $slug = $this->property('slug');
        $post = BlogPost::isPublished()->where('slug', $slug)->firstOrFail();
        /*
         * Add a "url" helper attribute for linking to each category
         */
        if ($post->categories->count()) {
            $post->categories->each(function($category){
                $category->setUrl($this->categoryPage, $this->controller);
            });
        }
        return $post;
    }
}
`

В этом коде уже нет необходимости проверок на существование объекта `$post`. Если объекта не будет, то текущая работа прервется и будет показана страница 404 ошибки. Фактически, мы переложили проверку на пустоту объекта на стандартные механизмы ORM. Теперь мы можем удалить и из всех остальных мест проверку на существование объекта.

Использование исключений позволяет упростить код и привести его к более понятному виду. Для того чтобы начать использовать исключения часто не нужно много усилий, современные средства и пакеты ориентированны именно на это. 