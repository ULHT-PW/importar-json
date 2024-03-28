# Importação de dados em ficheiros JSON na BD duma webapp Django

* Neste documento descrevem-se os passos para importar dados em JSON numa aplicação web.
* aplicação exemplo:
    * bibliotecaluso.pythonanywhere.com/admin
    * admin, admin
* [video demo](https://educast.fccn.pt/vod/clips/1oye8xaocv/link_box_h?locale=pt) pode ver todos os passos desde a criação da aplicação até a BD criada  

## 1. Ficheiros JSON

* Crie os ficheiros json com a informação que pretende carregar. 
* Guarde-nos na aplicação django para onde os quer importar, por exemplo numa pasta /json.
* Analise com atenção a informação que tem disponivel, e os tipos de dados que a armazena (lista, dicionario, string, inteiro, etc)
  
autores.json
```json
{
  "Machado de Assis": {
    "nacionalidade": "Brasileiro",
    "ano_nascimento": 1839,
    "ano_falecimento": 1908
  },
  "Eça de Queirós": {
    "nacionalidade": "Português",
    "ano_nascimento": 1845,
    "ano_falecimento": 1900
  }
}
```

livros.json
```json
[
  {
    "titulo": "Memórias Póstumas de Brás Cubas",
    "autor": "Machado de Assis",
    "genero": "Romance",
    "ano_publicacao": 1881,
    "lingua": "Português"
  },
  {
    "titulo": "O Primo Basílio",
    "autor": "Eça de Queirós",
    "genero": "Romance",
    "ano_publicacao": 1878,
    "lingua": "Português"
  }
]
```
## 2. Definição de models

Crie modelos que vão o encontro do que pretende.

```python
# models.py

from django.db import models

class Autor(models.Model):
    nome = models.CharField(max_length=50)
    ano_nascimento = models.IntegerField()
    nacionalidade = models.CharField(max_length=50)

    def __str__(self):
        return self.nome


class Livro(models.Model):
    titulo = models.CharField(max_length=50)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    genero =  models.CharField(max_length=50)
    ano_publicacao = models.IntegerField()


    def __str__(self):
        return f"{self.titulo} ({self.autor})"
```

# 3. Crie script para carregamento

* Crie na pasta da sua aplicação um script para carregar a informação dos json na base de dados
* Analise com atenção a informação que tem disponivel nos ficheiros, e os tipos de dados que a armazena (lista, dicionario, string, inteiro, etc), de modo a extrair corretamente a informação e criar os objetos correspondentes.

```python
# loader.py

from biblioteca.models import *
import json

Autor.objects.all().delete()
Livro.objects.all().delete()

with open('biblioteca/json/autores.json') as f:
    autores = json.load(f)

    # autores é um dicionario de autores, tendo como
    # chave o nome do autor, e como
    # valor um dicionario com informação do autor  

    for autor, info in autores.items(): 
        Autor.objects.create(
            nome = autor,
            ano_nascimento = info['ano_nascimento'],
            nacionalidade = info['nacionalidade']
        )

with open('biblioteca/json/livros.json') as f:
    livros = json.load(f)

    # livros é uma lista de dicionarios,
    # cada dicionario tendo informacao de um livro

    for livro in livros:
        Livro.objects.create(
            titulo = livro['titulo'],
            autor = Autor.objects.get(nome = livro['autor']),
            genero = livro['genero'],
            ano_publicacao = livro['ano_publicacao']
            )
```

## 4. execute na shell o script

Execute na shell do django o script. Para tal, na shell, importe o script.

```bash
13:08 ~/project $ python manage.py shell

Python 3.10.5 (main, Jul 22 2022, 17:09:35) [GCC 9.4.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.3.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from biblioteca import loader
```
