# Informações para gerar a Skill de Verify

Quero que você crie uma skill com multiplas skills que será responsavel por varrer o projeto que eu fizer com IA e verificar e aplicar as validações e fundações a seguir, para garantir que o projeto estará bem estruturado, otimizado, escalavel e seguro. Essa skill tem que ser aginostica a tecnologia. 

### Padrão
- Basico que funciona, evitar o maximo colocar tecnlogia desnecessaria. 
- Codigo limpo e organizado, codigo em ingles, comentarios em portugues (Comentários com critério).  
- Cada classe com sua responsabilidade. 
- Clean Code:Funções pequenas, DRY (Don't Repeat Yourself), KISS (Keep It Simple, Stupid), Tratamento de erros claro, Coesão e baixo acoplamento.

### Fundamentos de Engenharia
Existe a possibilidade de alguns dos fundamentos abaixo não se encaixar no projeto no qual essa skill estará rodando, então tenha uma central na qual identifique quais os fundamentos são importantes e necessarios aplicar no projeto. 

#### Race Condition 
- Constranint única, operação atômica, lock da linha
- Ache todo lugar que lê e depois grava
- Explique o que acontece com 2 requisições juntas
- Corrige no banco, na ordem de preferencia
- Escreve o teste com 2 requisições simultâneas 

#### Idempotência
- Confere a assinatura do webhook
- Tabela de eventos com o ID como chave única 
- Idempotency key em toda cobrança que você faz
- Teste: mesmo evento 2 vezes e 2 vezes ao mesmo tempo

#### Nunca confie no cliente
- Recalcular tudo do lado do backend (Validação no front é educação com o usuário. Quem decide é o servidor)
- Regra de negocio é sempre no backend. Leve a regra de negocio do front para o servidor. 
- Chaves que não pode ir pro front: supabase service_role/secret, token de api, qualquer api paga. NEXT_PUBLIC ou VITE isso é publico o .env não esconde. Procure chave secreta e diga quais trocar
- Cada usuário, só com as linhas dele.
- Explique cada policy do banco em portugues. 
- Relatorio com o risco de cada achado

#### Separação de responsabilidade
- Mapeia o arquivo e põe cada pedaço na gaveta
- Escreve testes do comportamento de hoje
- Plano em passos pequenos, app sempre de pé
- Roda os testes a cada passo.

#### Teste de integração
- A frase vem antes do teste
- IA mexeu? roda os testes
- Lista os fluxos critios: cadastro, checkout, webhook 
- Escreve a frase de cada comportamento e espera o meu OK
- Testa batendo na rota, com o banco de teste  mock só do que é de fora
- Me dá 1 comando para rodar tudo
- Regra: proibido apagar, pular ou afrouxar teste para fazer passar. 

### Ordem para rodar  
Teste de integração ->  Nunca confie no cliente -> Race Condition -> Idempotência -> Separação de responsabilidade
