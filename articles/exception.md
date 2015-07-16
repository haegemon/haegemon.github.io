Исключения - одна из тех тем которые не пользуются популяностью в php. Причина этого возможно в том, что  без исключений можно обойтись. Кроме того хотя исключения появились в версии 5, но многие продолжают писать код как будто он написан для php4. Однако исключения позволяют уменьшить количество условных операторов в вашей программе и как следствие уменьшить сложность самого кода. Кроме этого основная проблема с функциями, которые работают без исключений, что необходимо всегда держать в памяти и обрабатывать все состояния, которые могу вернуться. Неплохим примером будет небольшой рефакторинг который я недавно делал.

Как я писал для своего блога я использую октоберЦМС. Когда я использую программу с открытым исходным кодом, то я смотрю его тикеты, чтобы понимать, что от него можно ожидать. Увидев один тикет я был очень смущен:
> Doesn't show 404 page when post is not exists

Я попробовал повторить это у себя в блоге и получил 500 ошибку:
[screenshot with 500 error]

Когда я заглянул в исходный код проекта, то увидел в чем дело:

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

То есть если объект `$post` найден, то он передастся дальше, а если не найден, то он опять же передастся дальше. Кроме того стоит обратить внимаение на условие. В нем проверяется есть ли объект `$post`. Но если его нет, то ничего не происходит. Теперь если вспомнить какой код советуют написать разработчики на самой странице:

`
<?php
function onEnd()
{
        $this->page->title = $this->post->title;
}
?>
`

То становится понятно откуда появилась 500 ошибка: пост не найден, но система пытается задать имя для страницы все равно.

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
Стоит опять обратить внимание на условный оператор. Мы уже в двух местах видим проверку на существование объекта `$post`. Можно ли как то улучшить код, чтобы убрать эти проверки: да, для этого сразу после того как мы нашли объект `$post`. Мы должны проверить, что объект существует, и, если это не так, то прервать текущие действия и перейти на страницу 404 ошибки.

В laravel встроенны прекрасные методы работы с объектами, которые могут вызывать исключения если объект не найден. Использую их можно переписать код нахождения страницы:

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

В этом коде уже нет необходимости проверок на существование объекта `$post`. Если объекта не будет, то текущая работа прервется и будет показана страница 404 ошибки. Фактически мы переложили эти проверки на стандартные механизмы ORM, что помогает нам упростить код. Кроме этого мы теперь можем удалить и из всех остальных мест проверку на существование объекта.