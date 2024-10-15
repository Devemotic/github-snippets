### Entretien technique - Frontend - 1

Peux-tu citer au moins 3 "problèmes" dans le snippet ci-dessous ? On se basera sur les hypothèses suivantes :

-   l'indentation / la syntaxe est supposée correcte
-   pour des raisons de simplicité, tout est regroupé dans un unique fichier, mais en conditions réelle nous aurons des fichiers séparés (template HTML, code du composant, code de service Angular)

Le composant a un fonctionnel volontairement limité :

-   lister des articles de blog venant du backend
-   créer un article avec un titre uniquement

```ts
import { HttpClient } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
    selector: 'my-component',
    template: `
        <ul>
            <li *ngFor="let article of articles" (click)="goTo(article.id)">
                {{ article.titre }}
            </li>
        </ul>
        <div>
            <input type="text" [(ngModel)]="titre" />
            <button (click)="creerArticle()">Créer un article</button>
        </div>
    `,
})
export class MyComponent implements OnInit {
    private articles: any;
    private titre: string;

    constructor(private httpClient: HttpClient, private router: Router) {}

    ngOnInit(): void {
        this.httpClient
            .get('http://localhost:8080/api/articles')
            .subscribe((articles) => {
                this.articles = articles;
            });
    }

    goTo(id: string) {
        this.router.navigateByUrl('/articles/' + id);
    }

    creerArticle() {
        this.httpClient
            .post('http://localhost:8080/api/articles', { titre: this.titre })
            .subscribe((article) => {
                this.articles.push(article);
            });
    }
}
```
