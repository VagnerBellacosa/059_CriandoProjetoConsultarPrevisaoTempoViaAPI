#### WEB

# Autenticação de Web API com API Key

![Luiz Duarte](https://www.gravatar.com/avatar/93a9abccc4d85eefdb90b62bd810342e?d=mm&s=128)

Escrito por [**Luiz Duarte**](https://www.luiztools.com.br/post/author/luiztools/)em 02/09/2020

##### JUNTE-SE A MAIS DE 22 MIL PROFISSIONAIS DE TI

### Entre para minha lista e receba conteúdos exclusivos e com prioridade

Quando o assunto são web APIs, certamente você não irá querer permitir clientes anônimos, certo?

Identificar corretamente quem está fazendo as chamadas não apenas impede o acesso anônimo como restringe a autorização a determinadas ações e permite tomar medidas administrativas como controle de volumetria, estatísticas, revogação de acesso e muito mais.

Com tudo isso em mente, existem diversas formas de autenticação disponíveis para Web APIs, sendo API Keys uma delas, [JWT](https://www.luiztools.com.br/post/autenticacao-json-web-token-jwt-em-nodejs/) outra e [OAuth](https://www.luiztools.com.br/post/guia-oauth-para-iniciantes-epico/) uma terceira igualmente popular.

A ideia do artigo de hoje não é só ensinar como fazer autenticação via API Key como também explicar um pouco mais sobre segurança de Web APIs. Em outra oportunidade, eu ensino a programá-las.

Veremos aqui:

- [Formas de autenticação de Web APIs](https://www.luiztools.com.br/post/autenticacao-de-webapi-com-api-key/#1)
- [Por que usar API Keys em Web APIs?](https://www.luiztools.com.br/post/autenticacao-de-webapi-com-api-key/#2)
- [Como ter API Keys seguras?](https://www.luiztools.com.br/post/autenticacao-de-webapi-com-api-key/#3)

Vamos lá!

### Formas de Autenticação de Web APIs

Antes de avançar no tutorial de autenticação com API Key em Node.js, eu queria lhe explicar primeiro a diferença entre estas três, para que você consiga tomar a decisão correta para a sua aplicação.

**API Keys** são uma forma fácil e simples de fornecer acesso seguro de um sistema a uma Web API. Elas identificam o projeto ou sistema que está fazendo a requisição e sua duração é geralmente até ser revogada. Ou seja, se eu vou criar uma API de previsão do tempo que vai ser usada por outros sistemas, eu posso controlar o acesso por uma API Key por sistema (independente do número de usuários em cada um). Não importa nesse caso qual usuário está chamando, mas qual sistema sim.

**JSON Web Tokens (JWT)** são uma forma de complexidade intermediária para fornecer acesso seguro de um usuário a uma Web API. Eles identificam o usuário ou cliente que está fazendo a requisição e sua duração é de poucos minutos, podendo ser atualizado (refresh). Ou seja, se eu vou criar uma API para o backend da aplicação que meus usuários/clientes vão usar (um mobile app ou [SPA](https://www.luiztools.com.br/post/tutorial-de-react-js-com-node-js/) por exemplo), eu posso controlar o acesso via JWT, sabendo exatamente qual usuário está usando o backend.

**OAuth** é um protocolo avançado para [delegar acesso seguro](https://www.luiztools.com.br/post/delegacao-de-acesso-e-suas-motivacoes/) de um usuário em um sistema para outro sistema que permite várias configurações. Eles identificam que um terceiro pode fazer chamadas em seu nome para um sistema no qual você é cadastrado, como quando você usa login via Facebook, por exemplo. A duração é até o acesso ser revogado pelo detentor original das credenciais, é como se fosse uma procuração que o usuário assina autorizando ao sistema fazer chamadas em seu nome.

Em todos estes casos você pode ajustar a granularidade de permissões, duração, etc mas em linhas gerais, o resumo acima ilustra bem e deve ser útil para você tomar decisões futuras.

Dito isso, nesse artigo vamos falar da primeira forma: API Keys!

### Por que usar API Keys em Web APIs?

Como muito bem explicado [neste artigo do Google](https://cloud.google.com/endpoints/docs/openapi/when-why-api-key), usamos API Keys quando queremos fornecer acesso aos nossos dados a partir de um sistema terceiro, sendo que todas chamadas são feitas em nome do sistema em si, sem distinção de “usuários” do mesmo.

Assim, se eu desenvolvo uma plataforma de logística e licencio o acesso à web API de cálculo de frete para uma startup de ecommerce, eu posso criar um API Key para eta startup usar, da forma que ela quiser, e vou cobrar unicamente dele o uso, sem me preocupar se esta startup tem vários usuários ou não.

Mas eu não poderia fazer isso usando tokens também?

Poderia. Mas fazer com API Keys é muito mais simples para quem faz e para quem usa pois são estáticas, simples, curtas e não expiram. Você faz a chamada informando a API Key e pronto, obtém a resposta, sem fluxos adicionais, sem refresh, etc.

No entanto, como nem tudo são flores, é bom conhecermos os problemas deste módulo, para ter certeza de que se adequa ao seu cenário de uso.

API Keys são vulneráveis. Ou seja, se ela for capturada ou descoberta (mais sobre isso adiante), um terceiro poderá fazer chamadas em seu nome indefinidamente, até que você descubra o golpe e revogue a mesma.

Então o primeiro passo é aprender a gerarAPI Keys seguras.

### Como ter API Keys seguras?

O **primeiro passo** para ter API Keys seguras é na **geração** delas.

Cada API Key deve ser única, aleatória e não permitir adivinhação. O uso de caracteres alfanuméricos e especiais são uma boa prática, da mesma forma que fazemos com nossas senhas.

O **segundo passo** para ter API Keys seguras está no **armazenamento** delas.

Uma vez que você gere a API Key de um sistema/projeto, você terá de armazenar ela para consulta futura, quando for fazer a autenticação de uma requisição. E este armazenamento NÃO PODE ser em texto plano, ele tem de ser em forma de hashing, assim como fazemos com armazenamento de senhas.

Isso impede que, mesmo que alguém consiga ter acesso à sua base de dados, que não será possível descobrir a API Key original dos seus clientes, pois hashes são processos irreversíveis.

O tradeoff desta abordagem é que você jamais vai poder fornecer novamente o acesso à API Key caso o cliente se esqueça ou perca ela. Nesse caso, ele terá de criar uma nova, geralmente revogando a antiga.

O **terceiro passo** é definir alguma **granularidade** para suas API Keys, nunca dando acesso total à web API a partir de uma única chave.

Por uma questão de segurança, é melhor definir escopos de permissões por API Key, como leitura, leitura/escrita, por endpoint ou recurso, etc.

O **quarto passo** é não passar elas na URL das chamadas, mas ao invés disso, usá-las no cabeçalho **Authorization**.

O [padrão proposto](https://tools.ietf.org/html/rfc7235) é o abaixo, onde independente do [verbo/method HTTP](https://www.luiztools.com.br/post/http-para-programadores-node-js/) utilizado, passamos no cabeçalho Authorization a API key que vamos usar neste request.

| 1    | Authorization: Apikey 1234567890abcdef |
| ---- | -------------------------------------- |
|      |                                        |

Passar no header HTTP é mais seguro pois expõe menos a sua API Key para os pilantras que podem querer roubá-la. Obviamente isto só vai funcionar se a sua Web API estiver rodando sob HTTPS, então, embora eu não tenha falado aqui, a dica número um de segurança para qualquer aplicação web (API ou não) é usar SSL/TLS para criptografar a comunicação entre cliente e servidor.

E o **quinto e último passo** é ter **auditoria** do uso das API Keys, ou seja, registrar as requisições realizadas com cada uma delas.

Com esses logs, é possível aplicar controle de volumetria, estatísticas de uso, melhorar o dimensionamento das suas Web APIs e, para quem cobra por chamada, é o mínimo que se espera que você tenha né!

### Conclusões

Enquanto que no início da programação web as API Keys eram tudo que tínhamos para prover alguma segurança aos nossos serviços, hoje isso não é mais verdade. Não apenas a tecnologia evoluiu como surgiram diversos protocolos e técnicas mais avançados que endereçam os problemas de autenticação e autorização de requests de maneira mais especializada e segura.

Ainda assim, é possível que você encontre cenários de uso onde uma API Key tenha um fit interessante e não raro você vai encontrar exemplos de grandes e famosas web APIs usando este modelo, como as [APIs do Google](https://cloud.google.com/apis/).

Usar a ferramenta certa para cada problema é uma das coisas que separa programadores “comuns” dos [excepcionais](https://www.luiztools.com.br/post/8-dicas-para-se-tornar-um-excelente-programador/).

Um abraço e até a próxima, quando espero ensinar como programar uma web API que use API Keys para autenticação.