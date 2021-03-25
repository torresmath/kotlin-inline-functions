# kotlin-inline-functions
Um verbete dos meus estudos de Kotlin explicando o uso do modificador inline em funções

<br>
<h1> Inline Functions </h1>
Apesar da flexibilidade e diminuição de verbosidade em um código funcional, há um preço a se pagar pelo uso de Higher Order Functions em Kotlin. 
<br>
Ao executar uma Lambda, por exemplo, durante o tempo de compilação o Kotlin compila essa função como uma classe que sobrescreve o método Invoke da classe base Function. 
<br>
Isso significa que cada Lambda invocada em tempo de execução é uma nova alocação na memória Heap da JVM. No código abaixo criamos uma extensão para Array<String> que recebe uma function (String) -> Unit como parâmetro.

```kotlin 
 fun overhead() {
    val stringArray: Array<String> = arrayOf("1", "2", "3", "4", "5")
    stringArray.mapStringArray{
        println("MAPPING STRING $it")
    }
}

fun Array<String>.mapStringArray(block: (String) -> Unit) {
    for (s in this) block(s)
}
```
  
<br> <br>
Ao compilar o código, o Kotlin converteria a função para algo próximo disso:

```kotlin
fun overhead() {
    val stringArray: Array<String> = arrayOf<String>("1", "2", "3", "4", "5")
    val printsStrings = MapStringArray {
        println("MAPPING STRING $it")
    }
    printsStrings.invoke(stringArray)
}

/**
 * Classe geradas em tempo de compilação
 */
class MapStringArray(val block: (String) -> Unit) : (Array<String>) -> Unit {
    override fun invoke(p1: Array<String>) {
        for (s in p1) {
            block(s)
        }
    }
}
```

Durante o Runtime, o Kotlin instanciará uma variável local da classe MapStringArray dentro do escopo onde a Lambda é invocada. 
<br>
**Isso pode ser especialmente prejudicial para a memória quando temos um código com muitos laços de repetição invocando uma cadeia de High Order Functions, estimulando a execução mais frequente do Garbage Collector**. Esse efeito colateral do uso de HOFs é conhecido como **Overhead**.
Para evitar este tipo de problema, podemos adicionar o modificador **inline** na função. Assim o Kotlin fará uma instrumentação durante a compilação para “injetar” o conteúdo de uma função diretamente no bloco de código que foi invocada, evitando a criação e instanciamento de novas classes.
<br>

 ```kotlin
 inline fun Array<String>.mapStringArray(block: (String) -> Unit) {
    for (s in this) block(s)
}

/**
 * Durante a compilação,
 * a mesma função com inline é adicionada ao método que a invocou
 */
fun inlineCompiling() {
    val stringArray: Array<String> = arrayOf("1", "2", "3", "4", "5")
    for (s in stringArray)
        println("MAPPING STRING $s")
}
```
<br> <br>
O melhor de tudo é que inline functions indexam outras funções lambda dentro delas durante a compilação, portanto qualquer HOF dentro de uma inline function que **não esteja marcada com o modificador noinline** também será compilada dinamicamente no código.

<br>
<br>
<h1> Referências: </h1>
<br>
https://www.baeldung.com/kotlin/inline-functions
<br>
https://kotlinlang.org/docs/inline-functions.html
