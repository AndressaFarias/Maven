Maven no Pipeline do Jenkins
## Maven no Jenkins
A **seção** de _tools_ é uma das várias seções que podemos adicionar no _pipeline_, o que afeta a configuração do restante do pipeline. (Veremos os outros, incluindo o _agent_, em postagens posteriores.) Cada entrada de ferramenta fará as alterações nas configurações, como atualização de PATH ou outras variáveis ​​de ambiente, para disponibilizar a ferramenta nomeada no pipeline atual. Também instalará automaticamente a ferramenta nomeada se ela estiver configurada para fazê-lo em "Gerenciando o Jenkins" → "Configuração global da ferramenta".


~~~groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    tools {                     //  [1] seção tool para adicionar as configurações de tool 
        maven 'Maven 3.3.9'     //  [2] Configure este pipeline para usar a versão do Maven correspondente ao "Maven 3.3.9". configurado em "Gerenciando o Jenkins" → "Configuração global da ferramenta
        jdk 'jdk8'              //  [3] Configure este pipeline para usar a versão do Maven correspondente a "jdk8" (configurada em "Gerenciando o Jenkins" → "Configuração global da ferramenta").
    }

    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' //[4] Eles mostrarão os valores das variáveis ​​de ambiente PATH e M2_HOME.
            }
        }

        stage ('Build') {
            steps {
                echo 'This is a minimal pipeline.'
            }
        }
    }
}
~~~

Quando executamos esse pipeline atualizado da mesma maneira que executamos o primeiro, vemos que o pipeline declarativo adicionou outro estágio chamado "Declarativo: instalação da ferramenta": Na saída do console, vemos que, durante esse estágio específico, o "Maven 3.3.9" é instalado e as variáveis ​​de ambiente PATH e M2_HOME são definidas:

## Executando uma compilação Maven
Por fim, executar uma compilação do Maven é trivial. A seção de _tools_ já adicionou Maven e JDK8 ao _PATH_, tudo o que precisamos fazer é chamar _mvn install_. Seria bom se eu pudesse dividir a compilação e os testes em estágios separados, mas o Maven é famoso por não gostar quando as pessoas fazem isso, então deixarei em paz por enquanto.

Em vez disso, vamos carregar os resultados da compilação usando o plug-in JUnit; no entanto, a versão que acabou de compilar, desculpe.

~~~groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    tools {
        maven 'Maven 3.3.9'
        jdk 'jdk8'
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }

        stage ('Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true install'   // [1] Chame "mvn", a versão configurada pela seção "tools" será a primeira no path.

            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml'        // [2] Se a construção do maven tiver êxito, arquive os relatórios de teste do JUnit para exibição na interface da web da Jenkins. Discutiremos a seção de postagem em detalhes na próxima postagem do blog.
                }
            }
        }
    }
}
~~~



# Referencias
* Declarative Pipeline for Maveeeeen Projects : https://jenkins.io/blog/2017/02/07/declarative-maven-project/
    * Adding Tools to Pipeline - https://jenkins.io/blog/2017/02/07/declarative-maven-project/#adding-tools-to-pipeline