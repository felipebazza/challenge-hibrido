# Challenge Híbrido - Felipe Alexandre Bazzanella

## Descrição
Este projeto foi criado utilizando Magento na sua versão Community 2.3.5, pelo único motivo de que, a partir da versão 2.4.x, não há mais a possibilidade de utilização do instalador do Magento utilizando o browser.

## Pré-requisitos de instalação
Existem alguns pontos que precisam ser vistos antes da instalação do Magento, para otimização do tempo e assertividade.

### Servidor Web
É necessário configurar um servidor web, com PHP e MySQL para que o Magento funcione. Nessa versão, 2.3.5, é necessário ter o PHP nas versões 7.1, 7.2 ou 7.3 . Para criar esse projeto foi utilizado o XAMPP, em ambiente Windows 10. 

### Configurando o PHP
Independente do servidor web que for utilizado, é necessário entrar no arquivo php.ini, editando-o com o bloco de notas, e habilitando alguns módulos para que o composer trabalhe.
Para habilitar algum módulo basta encontrá-lo e deletar o ";" no começo da linha.
Módulos:
1. extension=intl
2. extension=soap
3. extension=sockets
4. extension=xsl

Estando nesse arquivo ainda, altere o parâmetro *memory_limit* para 3G. Se não for feito isso, o composer dará erro também.

### Composer
Como utilizei Windows, baixei o arquivo executável do site [Composer](https://getcomposer.org/download/). O detalhe aqui é que o Magento 2 não suporta ainda o Composer 2, somente o 1. Para resolver isso, após instalar o composer bastar abrir o terminal de comando e digitar 
``` composer selfupdate --1 ```

### MySQL
Achei melhor criar o banco manualmente. Para isso, basta ir até o XAMPP, na linha do MySQL clicar em "admin". Isso abrirá o PHPMyAdmin, onde, na esquerda, clica-se em Novo, dá-se um nome ao novo banco e Ok.

### Pasta
Criar uma pasta nova dentro da instalação do XAMPP/htdocs/, pois é onde ficarão os arquivos do Magento

## Instalação
### Baixando o Magento
Navegue até a pasta criada no último ponto acima, abra um terminal nela e digite 
``` composer create-project --repository=https://repo.magento.com/ magento/project-community-edition:2.3.5 . ```
Isso fará com que o composer baixe e instale todas as dependências do Magento. Esse processo deve rodar sem problemas, pois todos que poderiam ocorrer já foram revistos nos pontos da pré-instalação.

### Pré-instalação no Windows
Como utilizei Windows, é necessário realizar dois procedimentos para que o Magento rode sem problemas, devido a questão das barras no Windows serem diferentes das do Linux.

### Procedimento 1
1. Na pasta do Magento, navegue até a pasta ``` vendor/magento/framework/Image/Adapter/ ```. 
2. Abra o arquivo Gd2.php em algum editor de código de sua preferência. 
3. Encontre a função *validateURLScheme* na linha 96.
4. Comente toda a função.
5. Cole o código abaixo e salve o arquivo.
6. 
``` 
private function validateURLScheme(string $filename) : bool
  {
      $allowed_schemes = ['ftp', 'ftps', 'http', 'https'];
      $url = parse_url($filename);
      if ($url && isset($url['scheme']) && !in_array($url['scheme'], $allowed_schemes) && !file_exists($filename)) {
          return false;
      }

      return true;
  }
```

### Procedimento 2
1. Na pasta do Magento, navegue até a pasta ``` vendor/magento/framework/View/Element/Template/File/ ```.
2. Abra o arquivo Validator.php em algum editor de código de sua preferência.
3. Encontre a linha 138, onde está o código ``` $realPath = $this->fileDriver->getRealPath($path); ``` e comente-a.
4. Cole o código ``` $realPath = str_replace('', '/', $this->fileDriver->getRealPath($path)); ``` em seu lugar.

### Instalando o Magento
Abra seu navegador e entre no endereço ``` localhost/pasta-magento/ ```. Se tudo correu corretamente, haverá uma tela do Magento para instalação. O primeiro passo é a checagem de pré-requisitos, e a segunda é a etapa de configuração do banco de dados. Basta completar as informações requeridas e iniciar a instalação.
Um ponto importante aqui é a seleção do idioma Português - Brasil na instalação, pois isso habilita a edição dos campos de endereço posteriormente.

### Pós-instalação
No terminal, rodar os seguintes comandos
1. ``` php bin/magento indexer:reindex ``` - Reindex
2. ``` php bin/magento sampledata:deploy ``` - Baixa e configura dados de exemplo
3. ``` php bin/magento module:enable --all ``` - Habilita todos os módulos
4. ``` php bin/magento setup:upgrade ``` - Necessário para o registro dos módulos
5. ``` php bin/magento setup:static-content:deploy pt_BR -f ``` - Faz deploy do conteúdo estático para o idioma pt_BR
6. ``` php bin/magento cache:flush ``` - Este e abaixo para limpeza de cache
7. ``` php bin/magento cache:clean ```

## Removendo dois campos de endereço
Basta ir no admin do magento, em Lojas -> Configuração -> Clientes -> Configurações do cliente -> Opções de Nome e Endereço -> Opção *Número de linhas em Endereço*. Setar para 1.

## Setando os nomes dos campos do checkout ao contrário
Encontrei uma forma mais fácil de fazer isso no próprio arquivo .csv de tradução do Magento, em ``` app/i18n/rafaelcg/language_pt_br/pt_BR.csv ```. Basta abrir esse arquivo, encontrar os campos desejados e digitá-los ao contrário.

Após realizar esse procedimento no arquivo, foi necessário rodar alguns comandos para que os campos fossem efetivamente alterados.
1. ``` php bin/magento indexer:reindex ```
2. ``` php bin/magento setup:static-content:deploy pt_BR -f ``` 
3. ``` php bin/magento cache:flush ```
4. ``` php bin/magento cache:clean ```

## Alterando o botão de próxima etapa para retornar ao carrinho
Na página de checkout, caso não se saiba qual arquivo alterar, é preciso habilitar os template hints. No admin, ir até Lojas -> Configurações -> Avançado -> Desenvolvedor -> Depuração.
Na opção Habilitar as dicas de modelo na visão de loja, selecionar Sim e salvar.

Vai ser necessário rodar os códigos abaixo novamente
1. ``` php bin/magento indexer:reindex ```
3. ``` php bin/magento cache:flush ```
4. ``` php bin/magento cache:clean ```

Agora, ao acessar qualquer página, vão aparecer os arquivos que estão sendo utilizados para renderizá-la. No caso do checkout, foi o arquivo 
``` vendor/magento/module-checkout/view/frontend/web/template/shipping.html ```

Abra esse arquivo com o editor de código de sua preferência, navegue até a linha 74, onde está o código
```
<button data-role="opc-continue" type="submit" class="button action continue primary">
    <span translate="'Next'" />
</button>
```

Comente esse botão e digite o código abaixo, alterando para sua pasta Magento.
```
<a class="button action continue primary" href="http://localhost/pasta-magento/checkout/cart/">
    <span translate="'Back to Cart'">
</a>
```

Abrir novamente o arquivo ``` app/i18n/rafaelcg/language_pt_br/pt_BR.csv ``` e adicionar na última linha uma nova tradução
``` "Back to Cart", "Voltar para o Carrinho" ```

Após isso, será necessário rodar os comandos abaixo novamente, para refletir a mudança no front.
1. ``` php bin/magento indexer:reindex ```
2. ``` php bin/magento setup:static-content:deploy pt_BR -f ``` 
3. ``` php bin/magento cache:flush ```
4. ``` php bin/magento cache:clean ```
