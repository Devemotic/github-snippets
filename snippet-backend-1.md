### Entretien technique - Backend - 1

Peux-tu écrire ici ce qu'affiche le snippet suivant ? (on supposera que la classe autour est bien formée et que les imports sont corrects)

```java
public static void main(String[] args) {

    LocalDate date = LocalDate.of(2017, 5, 19);
    System.out.println(date);

    date.plus(20, ChronoUnit.DAYS);
    System.out.println(date);

    date = date.plus(10, ChronoUnit.DAYS);
    System.out.println(date);
}
```
