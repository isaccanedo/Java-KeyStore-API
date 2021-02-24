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

```
KeyStore ks = KeyStore.getInstance("JKS");
ks.load(new FileInputStream("newKeyStoreFileName.jks"), pwdArray);
```

Se nossa JVM não suportar o tipo de armazenamento de chave que passamos, ou se não corresponder ao tipo de armazenamento de chave no sistema de arquivos que estamos abrindo, obteremos uma KeyStoreException:

```
java.security.KeyStoreException: KEYSTORE_TYPE not found
```

Além disso, se a senha estiver errada, obteremos uma UnrecoverableKeyException:

```
java.security.UnrecoverableKeyException: Password verification failed
```

### Armazenando entradas
No armazenamento de chaves, podemos armazenar três tipos diferentes de entradas, cada entrada com seu alias:

- Chaves simétricas (referidas como chaves secretas no JCE),
- Chaves assimétricas (referidas como chaves públicas e privadas no JCE), e
- Certificados confiáveis
Vamos dar uma olhada em cada um.

### Salvando uma chave simétrica
A coisa mais simples que podemos armazenar em um armazenamento de chaves é uma chave simétrica.

Para salvar uma chave simétrica, precisaremos de três coisas:

1 - um alias - é simplesmente o nome que usaremos no futuro para nos referir à entrada
2 - uma chave - que é envolvida em um KeyStore.SecretKeyEntry.
3 - uma senha - que é envolvida no que é chamado de ProtectionParam.

```
KeyStore.SecretKeyEntry secret
 = new KeyStore.SecretKeyEntry(secretKey);
KeyStore.ProtectionParameter password
 = new KeyStore.PasswordProtection(pwdArray);
ks.setEntry("db-encryption-secret", secret, password);
```

Lembre-se de que a senha não pode ser nula; no entanto, pode ser uma String vazia. Se deixarmos a senha nula para uma entrada, obteremos uma KeyStoreException:

```
java.security.KeyStoreException: non-null password required to create SecretKeyEntry
```

Pode parecer um pouco estranho precisarmos agrupar a chave e a senha em classes de wrapper.

Envolvemos a chave porque setEntry é um método genérico que pode ser usado para outros tipos de entrada também. O tipo de entrada permite que a API KeyStore trate-o de maneira diferente.
Encapsulamos a senha porque a API KeyStore oferece suporte a retornos de chamada para GUIs e CLIs para coletar a senha do usuário final. Confira o KeyStore.CallbackHandlerProtection Javadoc para mais detalhes.

Também podemos usar este método para atualizar uma chave existente. Só precisamos chamá-lo novamente com o mesmo alias e senha e nosso novo segredo.

### Salvar uma chave privada
O armazenamento de chaves assimétricas é um pouco mais complexo, pois precisamos lidar com cadeias de certificados.

Além disso, a API KeyStore nos fornece um método dedicado chamado setKeyEntry, que é mais conveniente do que o método setEntry genérico.

Portanto, para salvar uma chave assimétrica, precisaremos de quatro coisas:
1- um alias, o mesmo de antes
2 - uma chave privada. Como não estamos usando o método genérico, a chave não será agrupada. Além disso, para nosso caso, deve ser uma instância de PrivateKey
3 - uma senha para acessar a entrada. Desta vez, a senha é obrigatória
4 - uma cadeia de certificação que certifica a chave pública correspondente

```
X509Certificate[] certificateChain = new X509Certificate[2];
chain[0] = clientCert;
chain[1] = caCert;
ks.setKeyEntry("sso-signing-key", privateKey, pwdArray, certificateChain);
```
Agora, muitos podem dar errado aqui, é claro, como se pwdArray for null:

```
java.security.KeyStoreException: password can't be null
```

Mas, há uma exceção muito estranha para se estar ciente, e é se pwdArray for um array vazio:

```
java.security.UnrecoverableKeyException: Given final block not properly padded
```

Para atualizar, podemos simplesmente chamar o método novamente com o mesmo alias e uma nova privateKey e certificateChain.

Além disso, pode ser valioso fazer uma rápida atualização sobre como gerar uma cadeia de certificados.

### Salvando um certificado confiável
Armazenar certificados confiáveis é bastante simples. Requer apenas o alias e o próprio certificado, que é do tipo Certificado:

```
ks.setCertificateEntry("google.com", trustedCertificate);
```

Normalmente, o certificado é aquele que não geramos, mas veio de um terceiro.

Por isso, é importante observar aqui que o KeyStore não verifica realmente esse certificado. Devemos verificar por conta própria antes de armazená-lo.

Para atualizar, podemos simplesmente chamar o método novamente com o mesmo alias e um novo trustedCertificate.

### Lendo inscrições
Agora que escrevemos algumas entradas, certamente vamos querer lê-las.

### Ler uma única entrada
Primeiro, podemos extrair chaves e certificados por meio de seu alias:

```
Key ssoSigningKey = ks.getKey("sso-signing-key", pwdArray);
Certificate google = ks.getCertificate("google.com");
```

Se não houver uma entrada com esse nome ou se for de um tipo diferente, getKey simplesmente retornará nulo:

```
public void whenEntryIsMissingOrOfIncorrectType_thenReturnsNull() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"

   Assert.assertNull(ks.getKey("some-other-api-secret"));
   Assert.assertNotNull(ks.getKey("widget-api-secret"));
   Assert.assertNull(ks.getCertificate("widget-api-secret")); 
}
```

Mas, se a senha da chave estiver errada, obteremos o mesmo erro estranho de que falamos anteriormente:

```
java.security.UnrecoverableKeyException: Given final block not properly padded
```

### Verificando se um keystore contém um alias
Como o KeyStore apenas armazena entradas usando um mapa, ele expõe a capacidade de verificar a existência sem recuperar a entrada:

```
public void whenAddingAlias_thenCanQueryWithoutSaving() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"
```

```
assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.containsAlias("some-other-api-secret"));
}
```

### Verificando o tipo de entrada
Ou o KeyStore # entryInstanceOf é um pouco mais poderoso.
É como containsAlias, exceto que também verifica o tipo de entrada:

```
public void whenAddingAlias_thenCanQueryByType() {
    // ... initialize keystore
    // ... add a secret entry called "widget-api-secret"
```

```
assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.entryInstanceOf(
      "widget-api-secret",
      KeyType.PrivateKeyEntry.class));
}
```

### Excluindo entradas
O KeyStore, é claro, oferece suporte à exclusão das entradas que adicionamos:

```
public void whenDeletingAnAlias_thenIdempotent() {
    // ... initialize a keystore
    // ... add an entry called "widget-api-secret"
```

```
assertEquals(ks.size(), 1);
```

```
 ks.deleteEntry("widget-api-secret");
    ks.deleteEntry("some-other-api-secret");
```

```
 assertFalse(ks.size(), 0);
}
```

Felizmente, deleteEntry é idempotente, então o método reage da mesma forma, quer a entrada exista ou não.

### Excluindo um Keystore
Se quisermos excluir nosso armazenamento de chaves, a API não nos ajudará em nada, mas ainda podemos usar Java para fazer isso:

```
Files.delete(Paths.get(keystorePath));
```

Ou, como alternativa, podemos manter o armazenamento de chaves e apenas remover as entradas:

```
Enumeration<String> aliases = keyStore.aliases();
while (aliases.hasMoreElements()) {
    String alias = aliases.nextElement();
    keyStore.deleteEntry(alias);
}
```

### Conclusão
Neste artigo, falamos sobre o gerenciamento de certificados e chaves usando a API KeyStore. Discutimos o que é um keystore, como criar, carregar e excluir um, como armazenar uma chave ou certificado no keystore e como carregar e atualizar entradas existentes com novos valores.
