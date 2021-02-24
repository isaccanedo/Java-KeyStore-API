# Java-KeyStore-API
:star2: # Neste tutorial, estamos examinando o gerenciamento de chaves criptográficas e certificados em Java usando a API KeyStore.

### Keystores

Se precisarmos gerenciar chaves e certificados em Java, precisamos de um keystore, que é simplesmente uma coleção segura de entradas de alias de chaves e certificados.
Normalmente salvamos armazenamentos de chaves em um sistema de arquivos e podemos protegê-los com uma senha.
Por padrão, o Java tem um arquivo de armazenamento de chave localizado em JAVA_HOME/jre/lib/security/cacerts. Podemos acessar esse armazenamento de chaves usando a alteração de senha do armazenamento de chaves padrão.

Agora, com esse pano de fundo, vamos começar a criar nosso primeiro.

### Criação de um Keystore

Podemos criar facilmente um keystore usando o keytool ou podemos fazê-lo programaticamente usando a API KeyStore:

```
KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
```

Aqui, usamos o tipo padrão, embora existam alguns tipos de keystore disponíveis, como jceks ou pcks12.

Podemos substituir o tipo "JKS" padrão (um protocolo de armazenamento de chaves proprietário da Oracle) usando um parâmetro -Dkeystore.type:
```
-Dkeystore.type=pkcs12
```

Ou podemos, é claro, listar um dos formatos suportados em getInstance:
```
KeyStore ks = KeyStore.getInstance("pcks12");
```

### Inicialização

Inicialmente, precisamos carregar o keystore:
```
char[] pwdArray = "password".toCharArray();
ks.load(null, pwdArray);
```

Usamos load quer estejamos criando um novo keystore ou abrindo um existente.
E dizemos ao KeyStore para criar um novo, passando null como o primeiro parâmetro.
Também fornecemos uma senha, que será usada para acessar o keystore no futuro. Também podemos definir isso como null, embora isso tornasse nossos segredos abertos.

### Armazenar
Por fim, salvamos nosso novo armazenamento de chaves no sistema de arquivos:
```
try (FileOutputStream fos = new FileOutputStream("newKeyStoreFileName.jks")) {
    ks.store(fos, pwdArray);
}
```

Observe que não são mostradas acima as várias exceções verificadas que obtêm Instância, carregam e armazenam cada lançamento.

### Carregando um Keystore

Para carregar um keystore, primeiro precisamos criar uma instância de KeyStore, como antes.

Desta vez, porém, vamos especificar o formato, já que estamos carregando um existente:
