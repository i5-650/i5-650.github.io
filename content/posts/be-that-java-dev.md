---
title: "Le Java sur un projet conséquent"
date: 2024-04-01
draft: false
tags:
    - C
    - C++
    - Programming
    - Optimization
    - Teaching
    - Tips
categories:
    - Programming
---

## 🧑‍🏫 Cet article n'est pas une leçon 
Cet article n'a pas pour but d'être une leçon pour apprendre le Java mais plutôt des astuces ou règles à suivre pour un projet assez conséquent et avec beaucoup de règles métier.
Tout ce qui va être dit n'est pas forcément applicable dans tous les cas mais reste des bases assez saine pour une application grandissante en Java.

Si vous avez des informations complémentaires ou des corrections: n'hésitez pas à contribuer à cet article.

## 📏 Les règles de base
### 💊 Remplacer les `null` par des wrappers
Le `null` est un fléau en Java. Il est souvent source de `NullPointerException` et il est très facile d'oublier de vérifier la nullité d'un objet. Il est donc préférable de ne pas utiliser de `null` dans votre code.
On va donc préférer les wrappers. Les wrappers sont des classes qui enveloppent un autre type (Optional<T>, List<T>, etc.). Ils permettent de gérer les cas où la valeur est absente.

Cela permet aussi de fonctionnellement se poser la question de quoi faire si une donnée n'est pas présente.

#### 1️⃣ Valeur unique
```java
// on a la fonction suivante
Declarant findDeclarantIfExist(...);

// on l'utilise de la manière suivante
Declarant declarant = findDeclarantIfExist(...);
if (declarant != null) {
    // on fait quelque chose
}
```
Devient:
```java
// on a la fonction suivante
Optional<Declarant> findDeclarantIfExist(...);

// on l'utilise de la manière suivante
Optional<Declarant> declarant = findDeclarantIfExist(...);
if (declarant.isPresent()) {
    // on fait quelque chose
}
```

#### 📋 Liste de valeur
```java
List<Chose> getChoses(int param){
    if(param == 2){
        return null;
    }
    return this.choses;
}
```
Devient:
```java
List<Chose> getChoses(int param){
    if(param == 2){
        return Collections.emptyList();
    }
    return this.choses;
}
```
#### 📚 Map de valeur
```java
Map<String, Chose> getChoses(int param){
    if(param == 2){
        return null;
    }
    return this.choses;
}
```
Devient:
```java
Map<String, Chose> getChoses(int param){
    if(param == 2){
        return Collections.emptyMap();
    }
    return this.choses;
}
```

#### ⚠️ Attention
Un wrapper ne doit presque jamais encapsuler un autre wrapper (sauf dans le cas d'une Map qui peut être une Map de Map ou Map de List). Si vous avez un wrapper qui encapsule un autre wrapper, c'est que vous avez mal conçu votre code.

### 🧰 Utiliser les Utils
Les Utils sont des classes qui regroupes des méthodes statiques et très utiles ! Ça permet de ne pas réinventer la roue; encore une fois, voici quelques exemples:
```java
StringUtils.isEmpty(str); // vérifier la nullité et la longueur d'une chaine
StringUtils.isNotBlank(str); // vérifier la nullité et la lvaleur d'une chaine
CollectionUtils.isEmpty(list); // vérifier la nullité et la taille d'une liste
CollectionUtils.isNotEmpty(list); // vérifier la nullité et la taille d'une liste
MapUtils.isEmpty(map); // vérifier la nullité et la taille d'une map
MapUtils.isNotEmpty(map); // vérifier la nullité et la taille d'une map
RandomStringUtils.randomAlphabetic(10); // générer une chaine aléatoire de 10 caractères
RandomStringUtils.randomNumeric(10); // générer une chaine numérique aléatoire de 10 caractères
StringUtils.leftPad(str, 10, '0'); // ajouter des 0 à gauche d'une chaine
StringUtils.rightPad(str, 10, '0'); // ajouter des 0 à droite d'une chaine
... // il y en a beaucoup d'autres, je ne montre que les plus courants 
```

### 🌊 Le streaming
Le streaming est une fonctionnalité très puissante de Java. Elle permet de manipuler des collections de manière très simple et très lisible. Cependant, il faut garder à l'esprit que: 1 stream = 1 parcours de la liste. Il ne faut donc pas abuser des streams. Par exemple:
```java
List<CompositeItem> items = ...;

// Mauvaise pratique
List<SousItemA> sousListA = items.stream().map(CompositeItem::getSousItemA).collect(Collectors.toList());
List<SousItemB> sousListB = items.stream().map(CompositeItem::getSousItemB).collect(Collectors.toList());
...

// Bonne pratique
List<SousItemA> sousListA = new ArrayList<>();
List<SousItemB> sousListB = new ArrayList<>();
for(CompositeItem item : items){
    sousListA.add(item.getSousItemA());
    sousListB.add(item.getSousItemB());
}
```

### 🧮 Les conditions
Les conditions doivent être le plus clair possible. Il faut avoir un MAXIMUM de 3 conditions dans un `if`. Si vous avez plus de 3 conditions, il faut les factoriser. Par exemple:
```java
// todo
```

## 👑 Être compris par les autres
Avoir un code super performant et super optimisé est très satisfaisant. Mais si personne ne peut le reprendre ou le comprendre, il perd beaucoup en utilité. Idem pour des méthodes, fonctions, classes, etc. 

L'idée n'est pas d'avoir un code où vous vous retrouvez, mais un code où tout le monde se retrouve. Voici quelques règles qui peuvent aider à cela et faire de vous le collègue que tout le monde aime:

### 📝 Les commentaires
On dit souvent "un bon code se passe de commentaires", oui mais non. Dans des petits projets où sur des méthodes très courtes, c'est vrai. Mais sur des projets conséquents, il est préférable de mettre des commentaires. Les commentaires permettent de comprendre le pourquoi du comment. 
**Un commentaire doit expliquer le pourquoi et non le comment.**, le comment est dans le code. Par exemple:
```java
// on récupère le déclarant
Declarant declarant = findDeclarantIfExist(...);

// on récupère les choses
List<Chose> choses = getChoses(declarant.getId());

// r´cupère les choses EXEMPLE
List<Chose> chosesValides = choses.stream().filter(c -> {
    Code codeChose = c.getCode();
    if(codeChose == CodeChoseEnum.EXEMPLE){
        return true;
    }
    return false;
}).collect(Collectors.toList());

declarant.setChoses(chosesValides);
```
Dans notre cas, on récupère les choses exemples pour un déclarant. C'est très facile de voir le comment, le code est assez simple mais pourquoi ? Pourquoi on récupère les choses exemples ? C'est là que le commentaire intervient.

### 🧾 Les Règles Métier
les règles métiers sont les règles que le client veut que l'on respecte. Elles ont toutes un titre tel que: `RM 0010598 Exemple de RM`. 

Ce qui est très pratique, est de mettre ces titres de règle dans la JavaDoc de votre fonction, et d'essayer de respecter 1 fonction = 1 RM. Par exemple:
```java

/**
 * RM 0010597 Récupère les choses exemple à partir d'un déclarant
 * Récupère les choses exemples pour un déclarant
 * 
 * @param declarant le déclarant
 * @return les choses exemples
 */
public List<Chose> getChosesExemples(Declarant declarant){
    List<Chose> choses = getChoses(declarant.getId());

    // RM 0010598 Recupérer les choses d'exemple pour un article
    List<Chose> chosesValides = choses.stream().filter(c -> {
        Code codeChose = c.getCode();
        if(codeChose == CodeChoseEnum.EXEMPLE){
            return true;
        }
        return false;
    }).collect(Collectors.toList());

    return chosesValides;
}
```
Avoir une RM dans la JavaDoc de votre fonction vous dispense d'expliquer pourquoi vous avez fait telle ou telle chose. Cela permet de gagner du temps et de la lisibilité. La personne qui se pose la question "pourquoi" n'a qu'à aller voir la spec et en profite pour vérifier que le code et la spec concordent. 

## 🏭 Config Spring
Spring est omniprésent dans nos projets et certaines choses sont parfois compliqué à configurer. Voici quelques astuces pour bien configurer votre Spring.

### 🫘 les beans 
Les beans sont des objets que Spring va instancier et gérer pour vous. Il y a plusieurs façons de déclarer un bean mais on va se concentrer sur l'injection de bean.

#### 📦 Injection par constructeur
L'injection par constructeur est la meilleure façon d'injecter un bean. Cela permet de rendre votre code plus lisible et plus facile à tester. Par exemple:
```java
@Component
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(ServiceB serviceB){
        this.serviceB = serviceB;
    }
}
``` 

