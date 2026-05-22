# minicurso-passo-a-passo
Passo a Passo para os alunos acompanharem o desenvolvimento.


#### Configurando o settings.py


**Registrar o app `posts`:**

```python

INSTALLED_APPS = [

    ...

    'posts',  # adicionar aqui

]

```



**Configurar idioma, fuso horário e redirecionamentos de login:**

```python

LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Sao_Paulo'



LOGIN_REDIRECT_URL = '/feed/'

LOGOUT_REDIRECT_URL = '/accounts/login/'

```



**Configurar pasta de templates:**

```python

TEMPLATES = [

    {

        ...

        'DIRS': [BASE_DIR / 'templates'],

        ...

    }

]

```



---



### ⏱️ Bloco 3 — Models e Banco de Dados 


#### 3.1 Escrevendo o model Post 



**Arquivo:** `posts/models.py`



```python

from django.db import models

from django.contrib.auth.models import User





class Post(models.Model):

    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')

    content = models.CharField(max_length=280)

    created_at = models.DateTimeField(auto_now_add=True)



    class Meta:

        ordering = ['-created_at']



    def __str__(self):

        return f'{self.author.username}: {self.content[:50]}'

```


#### 3.2 Criando e aplicando as migrations 



```bash

# Django lê o models.py e gera o arquivo de migration

python manage.py makemigrations



# Django aplica a migration no banco SQLite

python manage.py migrate



# Opcional: ver o SQL gerado

python manage.py sqlmigrate posts 0001

```

---



### ⏱️ Bloco 4 — Views e URLs



#### 4.1 Criando a primeira View



**Arquivo:** `posts/views.py`



```python

from django.views.generic import TemplateView

from .models import Post





class LandingView(TemplateView):

    template_name = 'home.html'



    def get_context_data(self, **kwargs):

        context = super().get_context_data(**kwargs)

        context['recent_posts'] = Post.objects.select_related('author')[:2]

        return context

```

#### 4.2 Configurando as URLs



**Criar `posts/urls.py`:**

```python

from django.urls import path

from . import views



urlpatterns = [

    path('', views.LandingView.as_view(), name='landing'),

]

```



**Registrar no `twitter_clone/urls.py`:**

```python

from django.contrib import admin

from django.urls import path, include



urlpatterns = [

    path('admin/', admin.site.urls),

    path('accounts/', include('django.contrib.auth.urls')),

    path('', include('posts.urls')),

]

```




### Bloco 5 — Templates Django 



#### 5.1 Criar a pasta de templates e o base.html 



```bash

mkdir templates

```



**`templates/base.html`** — estrutura mínima para começar:

```html

<!DOCTYPE html>

<html lang="pt-br">

<head>

    <meta charset="UTF-8">

    <title>Twitter Clone</title>

    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">

</head>

<body>

    <nav class="navbar navbar-dark bg-dark">

        <div class="container">

            <a class="navbar-brand" href="{% url 'landing' %}">🐦 Twitter Clone</a>

            {% if user.is_authenticated %}

                <span class="text-white">{{ user.username }}</span>

            {% else %}

                <a href="{% url 'login' %}" class="btn btn-outline-light btn-sm">Entrar</a>

            {% endif %}

        </div>

    </nav>



    <div class="container mt-4">

        {% block conteudo %}{% endblock %}

    </div>

</body>

</html>

```


#### 5.2 Criar o home.html 



**`templates/home.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<h1>Bem-vindo ao Twitter Clone</h1>



{% if recent_posts %}

    <h3>Posts recentes</h3>

    {% for post in recent_posts %}

        <div class="card mb-2">

            <div class="card-body">

                <strong>{{ post.author.username }}</strong>

                <p>{{ post.content }}</p>

                <small class="text-muted">{{ post.created_at|date:"d/m/Y H:i" }}</small>

            </div>

        </div>

    {% endfor %}

{% else %}

    <p class="text-muted">Nenhum post ainda. Seja o primeiro!</p>

{% endif %}

{% endblock %}

```



- `{% extends %}` → herda o base.html

- `{% for post in recent_posts %}` → loop nos posts enviados pela View

- `{{ post.created_at|date:"d/m/Y H:i" }}` → filtro de formatação de data



#### 5.3 Aplicar as migrations de auth e testar 



```bash

python manage.py migrate   # garante tabelas de auth criadas

python manage.py runserver

```





#### 2.1 Criando a View de cadastro



**`posts/views.py`** — adicionar:

```python

from django.contrib.auth.forms import UserCreationForm

from django.urls import reverse_lazy

from django.views.generic.edit import CreateView





class SignupView(CreateView):

    form_class = UserCreationForm

    template_name = 'registration/signup.html'

    success_url = reverse_lazy('login')

```



**Criar `templates/registration/signup.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<div class="row justify-content-center">

    <div class="col-md-5">

        <h2>Criar conta</h2>

        <form method="post">

            {% csrf_token %}

            {{ form.as_p }}

            <button type="submit" class="btn btn-primary w-100">Cadastrar</button>

        </form>

        <p class="mt-2">Já tem conta? <a href="{% url 'login' %}">Entrar</a></p>

    </div>

</div>

{% endblock %}

```



**Criar `templates/registration/login.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<div class="row justify-content-center">

    <div class="col-md-5">

        <h2>Entrar</h2>

        <form method="post">

            {% csrf_token %}

            {{ form.as_p }}

            <button type="submit" class="btn btn-primary w-100">Entrar</button>

        </form>

        <p class="mt-2">Não tem conta? <a href="{% url 'signup' %}">Cadastrar</a></p>

    </div>

</div>

{% endblock %}

```


#### 2.2 Protegendo rotas com LoginRequiredMixin



```python

from django.contrib.auth.mixins import LoginRequiredMixin



class PostCreateView(LoginRequiredMixin, CreateView):

    ...

```


---

#### 3.1 Formulário de Post



**Criar `posts/forms.py`:**

```python

from django import forms

from .models import Post





class PostForm(forms.ModelForm):

    class Meta:

        model = Post

        fields = ['content']

        widgets = {

            'content': forms.Textarea(attrs={

                'placeholder': 'O que está acontecendo?',

                'rows': 3,

                'class': 'form-control',

                'maxlength': 280,

            })

        }

```



#### 3.2 Feed com ListView



**`posts/views.py`** — adicionar:

```python

from django.views.generic import ListView





class FeedView(LoginRequiredMixin, ListView):

    model = Post

    template_name = 'posts/feed.html'

    context_object_name = 'posts'

    paginate_by = 20



    def get_queryset(self):

        return Post.objects.select_related('author')

```



**Criar `templates/posts/feed.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<h2>Feed</h2>



{% for post in posts %}

    <div class="card mb-3">

        <div class="card-body">

            <strong>{{ post.author.username }}</strong>

            <p>{{ post.content }}</p>

            <small class="text-muted">{{ post.created_at|date:"d/m/Y H:i" }}</small>

            {% if post.author == request.user %}

                <a href="{% url 'post-delete' post.pk %}" class="btn btn-sm btn-danger float-end">Deletar</a>

            {% endif %}

        </div>

    </div>

{% empty %}

    <p>Nenhum post ainda.</p>

{% endfor %}

{% endblock %}

```



#### 3.3 Criando posts



**`posts/views.py`** — adicionar:

```python

from django.views.generic.edit import CreateView, DeleteView

from .forms import PostForm





class PostCreateView(LoginRequiredMixin, CreateView):

    model = Post

    form_class = PostForm

    template_name = 'posts/create.html'

    success_url = reverse_lazy('feed')



    def form_valid(self, form):

        form.instance.author = self.request.user  # define o autor antes de salvar

        return super().form_valid(form)

```



**Criar `templates/posts/create.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<h2>Novo post</h2>

<form method="post">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit" class="btn btn-primary">Publicar</button>

    <a href="{% url 'feed' %}" class="btn btn-secondary">Cancelar</a>

</form>

{% endblock %}

```



#### 3.4 Deletando posts com segurança 



**`posts/views.py`** — adicionar:

```python

class PostDeleteView(LoginRequiredMixin, DeleteView):

    model = Post

    template_name = 'posts/confirm_delete.html'

    success_url = reverse_lazy('feed')



    def get_queryset(self):

        return Post.objects.filter(author=self.request.user)  # só os próprios posts

```



**Criar `templates/posts/confirm_delete.html`:**

```html

{% extends "base.html" %}



{% block conteudo %}

<h2>Deletar post?</h2>

<p>"{{ object.content }}"</p>

<form method="post">

    {% csrf_token %}

    <button type="submit" class="btn btn-danger">Sim, deletar</button>

    <a href="{% url 'feed' %}" class="btn btn-secondary">Cancelar</a>

</form>

{% endblock %}

```



**Registrar todas as URLs em `posts/urls.py`:**

```python

urlpatterns = [

    path('', views.LandingView.as_view(), name='landing'),

    path('feed/', views.FeedView.as_view(), name='feed'),

    path('accounts/signup/', views.SignupView.as_view(), name='signup'),

    path('post/novo/', views.PostCreateView.as_view(), name='post-create'),

    path('post/<int:pk>/deletar/', views.PostDeleteView.as_view(), name='post-delete'),

]

```




#### 4.1 Adicionando o model Like 



**`posts/models.py`** — adicionar abaixo da classe Post:

```python

class Like(models.Model):

    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='post_likes')

    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='likes')

    created_at = models.DateTimeField(auto_now_add=True)



    class Meta:

        unique_together = ('user', 'post')  # 1 like por usuário por post — banco rejeita duplicatas

```



```bash

python manage.py makemigrations

python manage.py migrate

```



#### 4.2 A LikeView com toggle 



**`posts/views.py`** — adicionar:

```python

from django.views import View

from django.shortcuts import get_object_or_404, redirect

from .models import Post, Like





class LikeView(LoginRequiredMixin, View):

    def post(self, request, pk):

        post = get_object_or_404(Post, pk=pk)

        like, created = Like.objects.get_or_create(user=request.user, post=post)

        if not created:

            like.delete()  # toggle: já curtiu → descurte

        return redirect(request.META.get('HTTP_REFERER', '/feed/'))

```



- `get_or_create` → busca no banco; se não achar, cria. Retorna `(objeto, foi_criado?)`

- `created=True` → curtiu agora / `created=False` → já existia, então deletar

- `HTTP_REFERER` → volta para a página de onde veio (feed ou detalhe)



**Registrar URL:**

```python

path('post/<int:pk>/like/', views.LikeView.as_view(), name='post-like'),

```



#### 4.3 Botão de like no feed.html 



**No `templates/posts/feed.html`**, dentro do card de cada post:

```html

<form method="post" action="{% url 'post-like' post.pk %}">

    {% csrf_token %}

    {% with likes=post.likes.all %}

    <button type="submit" class="btn btn-sm {% if request.user in likes %}btn-danger{% else %}btn-outline-danger{% endif %}">

        ❤️ {{ post.likes.count }}

    </button>

    {% endwith %}

</form>

```



- `post.likes.all` → acesso reverso via `related_name='likes'` definido no model

- `request.user in likes` → verifica se o usuário logado já curtiu



---



### ⏱️ Bloco 5 — Django Admin 



#### 5.1 Criando superusuário

```bash

python manage.py createsuperuser

```

Acessar: `http://127.0.0.1:8000/admin/`



#### 5.2 Registrando os models no admin



**`posts/admin.py`:**

```python

from django.contrib import admin

from .models import Post, Like





@admin.register(Post)

class PostAdmin(admin.ModelAdmin):

    list_display = ['author', 'content', 'created_at']

    list_filter = ['created_at']

    search_fields = ['content', 'author__username']





@admin.register(Like)

class LikeAdmin(admin.ModelAdmin):

    list_display = ['user', 'post', 'created_at']

```



- `list_display` → colunas na listagem

- `search_fields` → campo de busca

- `list_filter` → filtros laterais

- Tudo gerado automaticamente — sem escrever HTML



---


#### 6.2 Onde continuar aprendendo



| Recurso | Foco |

|---------|------|

| docs.djangoproject.com | Documentação oficial |

| Django Girls Tutorial | Guia completo para iniciantes |

| CS50W — Harvard (YouTube) | Web com Django e JavaScript |

| Real Python | Artigos e tutoriais práticos |



- Fazer deploy gratuito no Railway ou Render

- Adicionar ao LinkedIn e GitHub como projeto de portfólio




## 📋 Material de Apoio



### Cheat Sheet Django — Comandos



| Comando | Descrição |

|---------|-----------|

| `django-admin startproject nome .` | Cria o projeto |

| `python manage.py startapp nome` | Cria um app |

| `python manage.py runserver` | Inicia servidor de desenvolvimento |

| `python manage.py makemigrations` | Gera migrations a partir dos models |

| `python manage.py migrate` | Aplica migrations no banco |

| `python manage.py createsuperuser` | Cria usuário admin |

| `python manage.py shell` | Shell Python com Django carregado |

| `python manage.py showmigrations` | Mostra status das migrations |

| `python manage.py sqlmigrate posts 0001` | Mostra o SQL de uma migration |



### URLs do projeto ao final do Dia 2



| URL | Descrição |

|-----|-----------|

| `/` | Landing page |

| `/feed/` | Feed de posts |

| `/accounts/signup/` | Registro |

| `/accounts/login/` | Login |

| `/accounts/logout/` | Logout |

| `/post/novo/` | Criar post |

| `/post/<id>/deletar/` | Deletar post |

| `/post/<id>/like/` | Curtir/descurtir |

| `/admin/` | Painel administrativo |



### Tags Django mais usadas



```html

{% extends "base.html" %}             {# herança de template #}

{% block conteudo %}...{% endblock %} {# área substituível #}

{% url 'feed' %}                      {# gera URL pelo nome #}

{% csrf_token %}                      {# proteção CSRF — obrigatório em forms POST #}

{% if user.is_authenticated %}        {# condicional #}

{% for post in posts %}               {# loop #}

{% empty %}                           {# caso a lista esteja vazia #}

{{ post.content }}                    {# exibe variável #}

{{ post.created_at|date:"d/m/Y" }}   {# filtro de formatação #}

{% with var=valor %}                  {# variável local no template #}

```


