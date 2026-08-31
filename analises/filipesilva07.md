# Autopsia: GitLab 2017

**Autor:** Filipe Rezende Silva (@filipesilva07)

**Fonte primaria:** GitLab. *Postmortem of database outage of January 31*. 10 fev. 2017.
https://about.gitlab.com/blog/postmortem-of-database-outage-of-january-31/

**Data de acesso:** 25/08/2026

## 1. O que aconteceu

Em 31 de janeiro de 2017, o GitLab.com sofreu uma interrupção grave causada pela remoção acidental de dados do banco de dados primário de produção.

Antes do incidente principal, o banco estava sofrendo um aumento de carga, associado a spam e a um processo que tentava remover dados de um usuário. Como consequência, a replicação entre o banco primário e o secundário ficou atrasada e acabou parando.

Na tentativa de reconstruir a replicação, um engenheiro precisava limpar o diretório de dados do banco secundário. Porém, a operação foi executada por engano no servidor primário. O processo foi interrompido rapidamente, mas aproximadamente 300 GB de dados já haviam sido removidos.

O GitLab.com ficou indisponível por muitas horas e parte dos dados de produção não pôde ser recuperada. A estimativa apresentada pelo GitLab foi de aproximadamente 5.000 projetos, 5.000 comentários e 700 novas contas de usuários afetados. Repositórios Git e wikis não foram perdidos porque estavam armazenados separadamente.

## 2. Por que a recuperação falhou

O problema mais grave não foi somente o erro humano, mas a combinação desse erro com mecanismos de recuperação que não funcionavam adequadamente.

O GitLab possuía backups periódicos utilizando `pg_dump`, armazenados no Amazon S3. Entretanto, esses backups estavam falhando porque o procedimento utilizava o PostgreSQL 9.2 enquanto o banco de produção utilizava o PostgreSQL 9.6.

Além disso, as notificações sobre a falha dos backups eram enviadas por e-mail, mas esses e-mails estavam sendo rejeitados devido a problemas relacionados ao DMARC. Dessa forma, a equipe não sabia que os backups estavam falhando.

Os snapshots de disco do Azure também não estavam habilitados para os servidores de banco de dados. A replicação existente não pôde ser utilizada para recuperação porque o banco secundário também havia sido afetado durante a tentativa de reconstrução.

Outro problema identificado foi a ausência de testes regulares de restauração. O procedimento de backup existia, mas não havia uma pessoa responsável por verificar regularmente se ele realmente poderia ser utilizado para recuperar o banco.

## 3. Qual foi a causa raiz

O postmortem utiliza a técnica dos "5 Whys" e separa o incidente em dois problemas: a indisponibilidade do GitLab.com e a demora para restaurá-lo.

A indisponibilidade ocorreu porque o diretório do banco primário foi removido acidentalmente enquanto a equipe tentava reconstruir o banco secundário. A situação foi precedida por problemas de carga e replicação e foi agravada pela existência de procedimentos manuais de recuperação que não eram suficientemente documentados ou automatizados.

A demora na recuperação ocorreu porque os mecanismos normais de backup não estavam funcionando. O backup com `pg_dump` falhava silenciosamente, os snapshots do banco não estavam disponíveis e o servidor secundário também não podia ser utilizado.

Assim, um erro operacional relativamente simples acabou provocando um impacto muito maior porque a infraestrutura não possuía mecanismos de recuperação suficientemente confiáveis.

## 4. O que poderia ter sido melhor

O principal aprendizado é que backup não significa apenas gerar arquivos de backup. É necessário monitorar sua execução e testar regularmente sua restauração.

O processo deveria ter identificado automaticamente a falha do `pg_dump` e alertado a equipe de forma confiável. Também seria importante possuir cópias independentes dos dados, snapshots dos servidores de banco e procedimentos de recuperação automatizados e testados.

A operação de produção também deveria depender menos de ações manuais. A documentação e os runbooks precisavam deixar mais claro em qual servidor uma operação estava sendo executada, reduzindo a possibilidade de executar comandos destrutivos no ambiente errado.

## 5. Relação com DevOps

O incidente demonstra que DevOps não se resume à automação de deploys. Práticas de DevOps também devem estar presentes na operação, observabilidade, recuperação e confiabilidade dos sistemas.

Nesse caso, automação poderia ter reduzido o risco de erro humano durante a reconstrução da replicação. Monitoramento poderia ter identificado que os backups estavam falhando. Testes automatizados de restauração poderiam ter revelado o problema antes de um incidente real.

O caso também demonstra a importância de definir responsabilidades. O próprio postmortem identificou a necessidade de atribuir um responsável pela durabilidade dos dados e de automatizar os testes de recuperação.

A principal lição para mim é que uma infraestrutura confiável não é aquela em que erros nunca acontecem, mas aquela em que os erros podem ser detectados rapidamente e o sistema consegue se recuperar deles com pouco impacto.