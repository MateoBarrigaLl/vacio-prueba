# Manual de instalacion 

Se debe instalar RAGnarōk - Local RAG for Copilot dentro de las extenciones de VsCode. 

![alt text](img/image.png)

Aplastamos ctrl+ship+p buscar "Preferencia: Abrir configuracion del area de trabajo (JSON)"

![alt text](img/image1.png)

Se nos despliega esta ventana y copiamos el contenido: 
![alt text](img/image2.png)

Vamos a la carpeta de .vscode para aplicar las configuraciones de nuestgitra ambiente, pegamos lo que anteriomene copiamos. 

Vamos a la parte de nuestras extenciones y abrimos RAG Topics 
![alt text](img/image3.png)

Ponemos en Crear un topic y agregamos el nombre del proyecto (qa-agent-regression-code) damos doble enter para poderlo crear el proyecto: 
![alt text](img/image-0.png)


Despues usamos CTRL+SHIP+P para buscar (RAG: Add GitHub Token) y escribimos: 
![alt text](img/image-1.png)
```text
github.com
```
![alt text](img/image-2.png)
Y aceptamos las configuracines en caso de que salgas. 

Ingresamos a github a sacar el token: 

![alt text](img/image-3.png)

Ingresamos credenciales y solo configuramos para repo:

![alt text](img/image-4.png)

Y le damos a generar token y lo copiamos para poder ponerlo en la parte del token. 

Despues de eso realizamos CTRL+SHIP+P e ingremos: 
```text
RAG: Add Git Repository to Topic.
```
![alt text](img/image-5.png)
Seleccionamos el repositorio de regression y despues seleccionamos el token que habiamos creado. 
![alt text](img/image-6.png)
![alt text](img/image-7.png)
Agregamos la ruta del repositorio git de donde queremos hacer las pruebas: 
![alt text](img/image-8.png)

Damos enter y despues ponemos el nombre de la rama donde se va usar. y por ultimo damos enter. 

Verificamos que extraiga los datos:
![alt text](img/image-9.png)

Nos vamos a la raiz del agente en la carpeta de .github ingresamos a standards y vemos el MD de rag-topic-config. Ingresamos y verificamos que el nombre del topic sea el mismo que el que psusimos en el RAG Topics:

![alt text](img/image-10.png)

Una ves verificado eso podemos realizar uno del agente por medio del Copilot de VsCode. 
