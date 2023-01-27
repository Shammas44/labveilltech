---
author: Sébastien Traber
title: Typescript Refactoring
date: 2023-01-25
math: true
---

Le refactoring est un processus qui consiste à améliorer la structure et la qualité du code sans altérer son comportement. <!--more--> Il peut s'agir de simplifier une logique complexe, de rendre le code plus lisible ou de corriger les erreurs de programmation.

Dans ce contexte, Typescript est un super outil pour refactorer son code. Typescript est un sur-ensemble de JavaScript qui ajoute des fonctionnalités telles que la typification et les classes pour améliorer la qualité du code.
Cela permet de détecter les erreurs de compilation à un stade précoce et de faciliter la maintenance du code.

Dans cet article, nous allons explorer quelques techniques de refactoring disponibles dans Typescript issue du livre Refactoring TypeScript aux éditions packt

# 1. Avoid Null check everywhere

## Empty Collections

Afin d'éviter d'avoir à vérifier si une collection n'est pas null, on peut lui assigner une valeur par défaut au moyen de la syntaxe suivante: `const x = condition || []`

```typescript
// ❌ wrong 
const legalCases: LegalCase[] = await fetchCasesFromApi()
for (const legalCase of legalCases){
    if(legalCase.documents != null){
        uploadDocuents(lealCase.documents)
    }
}

// ✅ correct
const fetchCasesFromApi = async function(){
    const legalCases: LegalCase[] = await fetch(url) || []
    for (const legalCase of legalCases){
        legalCase.documents = legalCase.documents || []
    }
    return legalCases
}
```

📚 Dans certains cas, au cas du typage, il n'est pas permis d'assigner un tableau vide comme valeur à une collection. On peut alors utiliser l'astuce suivante.

```typescript
class EmptyArray<T>{
    static create<T>(){
        return new Array<T>() 
    }
}
```

## Empty Object

Dans le cas suivant on considère un jeu vidéo comportant des niveaux, dont certains, contiennent un boss. Comment vérifier que le niveau courant contient un boss ?

```typescript
// ❌ wrong
if(currentLevel.boss != null){
    currentLevel.boss.fight(player) 
}

...

if(currentLevel.boss != null){
    currentLevel.completed = currentLevel.boss.isDead()
}
```

L'exemple suivant consiste à créer un objet représentant un __boss null__ de sorte que ce dernier fasse automatiquement gagner le joueur lorsque celui-ci est rencontré. Il n'est ainsi plus nécessaire de vérifier que le  boss est __non null__ qu'il soit instance de `Boss` ou `NullBoss` 

```typescript
// ✅ correct
interface Boss {
    fight(player:Player);
    isDead();
}

class Boss implements Boss {
    fight(player:Player){
        // do some logic and return boolean 
    }
    
    isDead(){
        // Return whether boss is dead depending on how fight went 
    }
}

class NullBoss implements Boss {
    fight(player:Player){
        // player always wins
    }
    
    isDead(){
        return true
    }
}
```

## Special Case Pattern 

Dans le cas du placement d'un ordre, ce dernier pourrait avoir plusieurs statuts qui nécessitent chacun un traitement différent.

__❌ Exemple incorrect__

```typescript
if (order.status === OrderStatus.Pending) {}
else if (order.status === OrderStatus.Canceled) {}
else if (order.status === OrderStatus.Terminated) {}
else {}
```

Afin d'éviter la situation précédente, on peut créer une version différente de la classe Order pour chaque Status.

__✅ Exemple correct__

```typescript

class PendingOrder implements Order {
    constructor(){}
    public placeOrder(){
        // API call
    }
}

class PaymentRejectedOrder implements Order {
    constructor(){}
    public placeOrder(){
        // Try to pay again 
    }
}

class OrderOnFraudulentAccount implements Order {
    constructor(){}
    public placeOrder(){
        // Notify the fraud departmment
    }
}
```

Quelque part dans le code, nous aurions une fonction qui créer une instance pour chaque ordre nécessitant d'être placé. puis chaque ordre sera traité différemment au moyen de leur méthode respective `placeOrder`

```typescript
const ordersCollection: Order[] = await getOrders();
for(const order of ordersCollection){
    order.placeOrder()
}
```

# 2. Wordy conditionals

l'astuce suivante permet d'éviter d'écrire des conditions trop complexes, imbriquant plusieurs conditions telles que : `if(x && y && z){}`

## Combining conditions

__❌ Exemple incorrect__

```typescript
if(user.role === "admin" 
&& user.active
&& user.permissions.some(p => p === "edit")){
    // do stuff
}
```

On peut faciliter la lecture en attribuant les conditions à des variables puis en regroupant toutes les conditions sous une seule variable avec un nom suffisamment descriptif.

📚 De manière générale, on peut faire en sorte que les `if statement` se réfère à une seule condition. S’il faut vérifier de multiples variables, on peut se permettre de les combiner en une seule variable portant un nom semantic clair. 

__✅ Exemple correct__

```typescript
const isAdmin: boolean = user.role === "admin";
const userIsActive: boolean = user.active;
const userCanEdit: boolean = user.permissions.some(p => p === "edit");

const activeAdminCanEdit: boolean = isAdmin && userIsActive && userCanEdit;
if(activeAdminCanEdit){
    // do stuff
}
```

## Extract methods from conditions

Dans l'exemple précédent, toutes les conditions se réfèrent à un seul entité, __l'utilisateur__, dans ce genre de situation, on peut extraire les conditions et les implémenter directement comme méthode à la classe `User`

```typescript
const activeAdminCanEdit: boolean = user.isAdmin() 
    && user.isActive()
    && user.canEdit();
if(activeAdminCanEdit){
    // do stuff
}
```

## SRP

Si le nombre de méthodes servant à vérifier une condition devient trop important dans une classe, on peut envisager de créer un class à part pour ce besoin selon le __Single Responsability Principle__

__SRP class__

```typescript
class UserIsActiveAdmin {
    private _user: User;
    
    constructor(user:User){
        this._user = user; 
    }
    
    public invoke(): boolean {
        return this._user.isAdmin() && this._user.isActive() 
    }
}
```

__SRP object usage__

```typescript
const activeAdminCanEdit: boolean = new UserIsActiveAdmin(user).invoke() && user.canEdit();

if(activeAdminCanEdit){
    // do stuff
}
```

## Pipe Classes

Si on applique le conseil précédent dans un cas qui nécessite de nombreuses conditions, on risque de se retrouver avec de nom de class très long et incompréhensible tels que `CheckOneCheckTwoCheckThreeCheckFour...` c'est pourquoi dans l'exemple suivant on va se limiter à vérifier _1 à 2_ conditions.

```typescript
interface PipeableCondition {
    check(): boolean;
}
```

```typescript
class UserIsActiveAdmin implements PipeableCondition {
    private _user: User;
    
    constructor(user: User){
        this._user = user; 
    }
    
    public check(): boolean {
        return this._user.isAdmin() && this._user.isActive();
    }
}
```

Si on dispose de nombre de ces conditions, on peut les combiner ensemble dans une collection et itérer dessus pour vérifier que toutes les conditions passent. 


```typescript
const conditions: PipeableConditon[] = [
    new UserIsActiveAdmin(user),
    new UserCanEdit(user),
    new UserIsNotBlacklisted(user),
    new UserLivesInAvailableLocation(user)
];

const valid = conditions.every(p => p.check());

if(valid){
    // do stuff
}
```

## Global Pipe Class

Si nous voulant utiliser l'exemple précédant partout dans le code il faut préparer une classe à cette intention.


__Declaratin de la classe ConditionsPipe__

```typescript
class ConditionsPipe {
    private _conditions: PipeableCondition[];
    
    constructor(conditions: PipeableCondition[]){
        this._conditions = conditions; 
    }
    
    check(): boolean {
        return this._conditions.every(p => p.check()); 
    }
}
```

__Utilisation de la classe ConditionsPipe__

```typescript
const pipe = new CondtionsPipe([
    new UserIsActiveAdmin(user),
    new UserCanEdit(user),
    new UserIsNotBlacklisted(user),
    new UserLivesInAvailableLocation(user)
])

if (pipe.check()){
    // do stuff
}
```

# 3. Nested Conditionals

Afin d'éviter d'avoir des conditions en cascades, rendant le code difficile à lire et à maintenir, on peut utiliser les conseils suivants

## Fail Fast

Le but et de faire échouer les tests le plus vites possible, pour ce faire on va notamment non pas vérifier si les conditions sont remplies, mais plutôt si elles ne le sont pas.

__❌ Exemple incorrect__

```typescript
let result = null;
 
if(!order.wasCancelled()){
    if(order.isPaid()){
        result = order.sendToShipping(); 
    }else {
        if(order.isFraudulent()){
            result = order.sendToFraudDept();
        }else{
            result = order.tryAgainLater(); 
        }
    }
}
return result;
```

__✅ Exemple correct__

```typescript
if(order.wasCancelled()){
    return; 
}
if(order.wasPaid()){
    return order.sendToShipping();
}
if(order.wasCancelled()){
    return;
}
if(order.wasPaid()){
    return order.sendToShipping();
}
if(order.isFraudulent()){
    return order.sendToFraudDept();
}
return order.tryAgainLater();
```

## Gate class

Ce pattern est utilisé lorsqu’on veut que'une partie d'une fonction de soit exécuté qui si elle remplit des conditions précises. 

```typescript
const accountIsVerified = await accountRepo.accountIsVerified(userAccount);
const canPlaceOrder = await orderRepo.userCanPlaceOrder(user);
if(accountIsVerified){
    if(canPlaceOrder){
        await orderRepo.placeOrder(order);
    }
}
```

Imaginons que dans l'exemple précédent nous ayons besoin que ce code soit utilisé à de multiples endroits dans l'application. Comment faire pour réaliser ces vérifications de manière plus explicite, partageable et facilement maintenable. la solution consiste à créer une `Gate`

On peut créer une `Gate` en extrayant chacune des conditions dans une nouvelle classe. ces classes lanceront une exception en cas d'échec

__Exemple de Gate__

```typescript
class AccountIsVerifiedGate {
    private _accountRepo: AccountRepo;
    
    constructor(accountRepo: AccountRepo){
        this._accountRepo = accountRepo; 
    }
    
    async invoke(account: UserAccount){
        const isVerified = await this._accountRepo.accountIsVerified(account);
        
        if(!isVerified){
            throw "Gate exception"; 
        }
    }
}
```

__Utilisation des Gates dans le code__

```typescript
await accountIsVerifiedGate.invoke(userAccount);
await userCanPlaceOrderGate.invoke(user);
await orderRepo.placeOrder(order);
```

Si un des tests échoue, la gate associée lance une exception. Il ne manquerait ainsi plus qu’à traiter les éventuelles exceptions.

📚 Ce genre de pattern peut se révéler très utile en ce qui consiste les appels à une API.

# 4. Primitive Overuse


__Exemple à améliorer__

```typescript
const domain: string = email.replace(/.*@/,"");
const userName = email.replace(domain, "");
const sendInternal: boolean = domain === "internal-compagny.com";
if(sendInternal){
    if(userName === "info"){
        mail.sendToCustomerServiceTeam(email, message); 
    }else{
        mailer.mailToInternalServer(email,message); 
    }
}else{
    throw "Cannot email externally";
}
```

__Amélioration 1__

```typescript
const domain: string = email.replace(/.*@/,"");
const userName = email.replace(domain, "");
const sendInternal: boolean = domain !== "internal-compagny.com";
if(sendExternal){
    throw "Cannot email externally";
}
if(userName === "info"){
    mailer.sendToCustomerServiceTeam(email,message);
    return;
}
mailer.mailToInternalServer(email, message);
```

Cependant on peut encore améliorer les conditions. Le principal souci est qu'il n'existe pas de concept d'adresse mail, pour y remédier, on peut créer une classe. `EmailAddress`

Dans le code précédant nous nous sommes focalisé sur la variable `string` contenant l'adresse mail d'ou émane toute la logique. À la place, on pourrait encapsuler ce concept et sa logique dans un nouvel objet.

__Create a value object__ 

```typescript
class EmailAddress {
    private _value: string;
    private _domain: string;
    private _userName: string;
    
    constructor(value: string){
        this._value = value; 
        this._domain: string = value.replace(/.*@/,"");
        this._userName = value.replace(domain, "");
        if(this.isExternal(value){
            throw "Cannot email externally";
        }
    }
    
    isExternal(): boolean {
        return this._domain !== "internal-company.com"; 
    }
    
    isInfoUser(): boolean {
        return this._userName === "info"; 
    }
    
    value(){
        return this._value; 
    }
}
```

📚 les `value object` ne doivent pas êtres modifié une fois instancié, on n'expose que les méthodes `read-only`qui donnent au reste de l'application des informations sur l'adresse mail.

__Utilisation du value object__

```typescript
try {
const emailAddress: EmailAddress = new EmailAddress(email);
doEmailLogic(emailAddress);
}catch(Exception e){
    // Do stuff
}

function doEmailLogic(emailAddress: EmailAddress):void {
    if(emailAddress.isInfoUser()){
        mailer.sendToCustomerServiceTeam(emailAddress.value(), message);
        return;
    }
    mailer.mailToInternalServer(emailAddress.value(), message);
}
```

## Deceptive Booleans

__❌ Exemple incorrect__

```typescript
const user: User = await userIsAuthenticated(username, password);
const isAuthenticated: boolean = user! == null;
if(isAuthenticated){
    if(user.isActive){
        redirectToUserDashboard(); 
    }else{
        returnErrorOnLoginPage("User is not active"); 
    }
}else{
    returnErrorOnLoginPage("Credentials are not valid.")
}
```

__✅ Exemple correct__

```typescript
const user: User = await userIsAuthenticated(username, password);
if(!user.isAuthenticatd()){
    returnErrorOnLoginPage("Credentials are not valid.")
}
if(!user.isActive()){
    returnErrorOnLoginPage("User is not active"); 
}
redirectToUserDashboard();
```

__Implémenter les fonctionnalités suivantes:__

- Si la session utilisateur existe déjà, rediriger l'utilisateur sur une page home spéciale.
- Si l'utilisateur a bloqué son compte dû à trop de tentatives de connexion, rediriger l'utilisateur sur une page spéciale.
- S’il s'agit de la première connexion de l'utilisateur, rediriger l'utilisateur vers une page de bienvenue.

__❌ Exemple incorrect__

```typescript
if(!user.isAuthenticated()){
returnErrorOnLoginPage("Credentials are not valid.");
}
if(!user.isActive()){
    returnErrorOnLoginPage("User is not active");
}
if(user.alreadyHadSession()){
    redirectToHomePage();
}
if(user.isLockedOut()){
    redirectToUserLockedOutPage();
}
if(user.isFirstLogin()){
    redirectToWelcomePage();
}
redirectToUserDashboard();
```

📚 Ce code est très lisible, mais à mesure que nous rajoutions des conditions nous surchargeons la classe `user` avec de la logique propre à l'authentification. on peut l'améliorer avec un `enum`

__✅ Exemple correct__

```typescript
enum AuthentificationResult {
    InvalidCredentials,
    UserIsNotActive,
    HasExistingSession,
    IsLockedOut,
    IsFirstLogin,
    Successful
}

const result: AuthentificationResult = await tryAuthentificateUser(username, password);
if(result === AuthentificationResult.InvalidCredentials){
    returnErrorOnLoginPage("Credentials are not valid");
}
if(result === AuthentificationResult.UserIsNotActive){
    returnErrorOnLoginPage("User is not active");
}
if(result === AuthentificationResult.HasExistingSession){
    redirectToHomePage();
}
if(result === AuthentificationResult.IsLockedOut){
    redirectToUserLockedOutPage();
}
if(result === AuthentificationResult.isFirstLogin){
    redirectToWelcomePage();
}
redirectToUserDashboard();
```

## Strategy Pattern

Ce bout de code est une amélioration du code précédent

```typescript
const strategies: any = [];
strategies[AuthentificationResult.InvalidCredentials] = () => {
    returnErrorOnLoginPage("Credentials are not valid");
}
strategies[AuthentificationResult.UserIsNotActive] = () => {
    returnErrorOnLoginPage("User is not active");
}
strategies[AuthentificationResult.HasExistingSession] = () => {
    redirectToHomePage();
}
strategies[AuthentificationResult.IsLockedOut] = () => {
    redirectToUserLockedOutPage();
}
strategies[AuthentificationResult.IsFirstLogin] = () => {
    redirectToWelcomePage();
}
strategies[AuthentificationResult.Successful] = () => {
    redirectToUserDashboard();
}
strategies[result]();

```

