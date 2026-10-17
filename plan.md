# Plano — Rede de barbearias

## Escopo e implementação

Manter o website existente React + Express + tRPC + Drizzle e transformá-lo em uma agenda multi-barbearia, sem reinicializar o projeto. O cliente escolhe uma barbearia no item **“Escolher barbearia”** do menu de três linhas; a lista pública mostra somente os nomes. Depois de autenticar como cliente, sua escolha fica vinculada à conta e os próximos agendamentos são criados para aquela loja.

A identidade autenticada que escolhe **“Iniciar sessão como cabeleireiro”** passa a receber o papel `barber` no callback OAuth, sem fila nem aprovação do administrador. Cada conta de cabeleireiro pode criar uma barbearia, informar seu nome e gerir apenas a própria loja. A opção **“Criar barbearia”** aparece quando ainda não existe loja para a conta; depois, o painel mostra os cortes agendados e os clientes que escolheram aquela loja. A lista de clientes inclui nomes mesmo quando estão offline e pode marcar quem está conectado recentemente.

A tabela `barbershops` guarda proprietário, nome e datas. `users.selectedBarbershopId` guarda a seleção do cliente. `appointments.barbershopId` vincula cada corte à loja. A migração aditiva cria uma loja inicial e associa a ela os agendamentos antigos e perfis cliente existentes, sem apagar dados. A loja histórica fica sem proprietário e fora da lista pública até o administrador OAuth designado reivindicá-la; nesse passo, o servidor preserva o antigo nome global configurado, quando houver. Assim, um cabeleireiro aleatório não herda o histórico. O papel do administrador continua reservado à identidade OAuth designada; esse papel não é concedido por escolher “barbeiro”.

O servidor expõe uma lista pública de lojas contendo somente ID e nome. Operações autenticadas verificam a propriedade da loja no servidor: cabeleireiros só listam/criam/confirmam/cancelam horários e consultam clientes da própria loja; clientes só fazem pedidos para a loja selecionada e veem seus próprios pedidos. A alteração do nome da loja só pode ser feita por seu proprietário autenticado (ou administrador designado). Não expor telefone/e-mail dos clientes na lista de lojas ou na lista de clientes escolhidos.

Manter as páginas públicas, fotos de serviços, temas, avatar, música, e o manifesto de rotas. As ações sobre agenda e perfil seguem o login OAuth existente; tornar o website público remove somente a barreira de acesso da plataforma e não cria automaticamente uma sessão de cliente. A visibilidade do site é uma configuração de compartilhamento do Dashboard, não uma variável da aplicação.

## Estrutura do projeto
- `client/src/pages/Home.tsx`: menu, seletor de barbearia, galeria e pedido para a loja selecionada.
- `client/src/pages/Login.tsx` e `client/src/const.ts`: escolha de acesso; OAuth retorna à tela interna do papel selecionado.
- `client/src/pages/Dashboard.tsx`: criação da loja e agenda/clientes do cabeleireiro, todos limitados à loja proprietária.
- `client/src/pages/Settings.tsx`: avatar, música, fundos, tema, edição do perfil e nome da própria loja.
- `client/src/App.tsx` e `client/src/index.css`: rotas, presença, preferências e componentes responsivos.
- `drizzle/schema.ts` e `drizzle/`: lojas e vínculo de clientes/agendamentos, com migração aditiva e compatibilidade dos dados atuais.
- `server/db.ts` e `server/routers.ts`: persistência e verificação de propriedade de loja em cada leitura/escrita.
- `server/_core/oauth.ts`, `server/_core/context.ts` e `server/_core/trpc.ts`: papel de cabeleireiro no login, sessão real e papel administrativo restrito.
- `client/public/manus-routes.json`: manifesto de páginas sincronizado com as rotas existentes.

## Direção visual
**Movimento:** oficina editorial urbana em carvão e cobre, mantendo a identidade já implementada.
**Princípios:** lojas identificáveis pelo nome; seleção rápida no menu móvel; áreas separadas por papel; legibilidade em celulares.
**Cores:** carvão para contraste, marfim para leitura e cobre como assinatura.
**Layout:** menu móvel de três linhas abre uma lista compacta de barbearias; telas internas separam a criação da loja, agenda e clientela.
**Assinaturas:** tesouras/máquinas como padrões sutis e marcadores numerados de serviços.
**Interação:** escolher loja confirma o contexto antes de reservar; cabeleireiro vê sempre o nome da própria loja; nenhuma ação de outro proprietário é executada por ID fornecido sem checagem de posse.
**Animação:** transições curtas, respeitando `prefers-reduced-motion`; sem autoplay.
**Tipografia:** títulos condensados para impacto e texto DM Sans para interface.
**Essência:** uma agenda simples para a clientela encontrar barbearias e reservar cortes; organizada, direta e acolhedora.
**Voz:** “Escolha sua barbearia.” / “Seu horário, do seu jeito.”
**Marca:** antes do login, o rótulo é “Gerencie sua barbearia”; após a sessão, exibe a loja escolhida pelo cliente ou a loja do cabeleireiro.
**Cor assinatura:** cobre quente.

## Suposições e limites registrados
Cada conta de cabeleireiro cria no máximo uma loja nesta versão. O cliente só agenda depois de completar o perfil de quatro campos e selecionar uma loja reivindicada. A loja inicial fica reservada ao administrador designado até que ele entre para reivindicá-la; então pode ser renomeada. Barbeiro é um papel de operação próprio da loja e não equivale ao papel de administrador. A publicação pública ainda respeita a autenticação OAuth do aplicativo para operações privadas. Não excluir dados legados nem compartilhar dados de clientes entre lojas.

O website e o aplicativo futuro não cobram dos clientes nem dos cabeleireiros e não incluem checkout, assinatura ou pagamentos. Essa decisão é sobre o produto e não promete que a hospedagem ou a publicação futura em uma loja de aplicativos sejam gratuitas.

## Correção visual solicitada — celular e Configurações

- A página inicial permanece dedicada à apresentação dos serviços e ao agendamento; o menu de três linhas começa fechado e leva à página Configurações. Foto, tema, música e fundos aparecem como controles somente em Configurações; retirar o botão rápido de tema do cabeçalho inicial e o player flutuante das demais páginas.
- Restaurar a folha global de estilos a partir da última versão completa do Git, preservando/reintegrando o seletor de barbearia, os cartões do cabeleireiro e os campos de perfil atuais. O documento mantém `width=device-width, initial-scale=1`; não aplicar zoom/escala forçada.
- No celular, manter o layout dentro da largura do dispositivo, sem rolagem horizontal, e usar fonte de ao menos 16 px nos campos editáveis para evitar o zoom automático do Safari ao focar um campo. Confirmar a prévia nas rotas `/` e `/configuracoes` a 375 × 812.
