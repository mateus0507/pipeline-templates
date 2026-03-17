# Define a imagem base usando Node.js versão 20 com Alpine Linux (imagem leve)
# "AS base" cria um estágio chamado "base" que será reutilizado nos próximos estágios
FROM node:20-alpine AS base

# Define o diretório de trabalho dentro do container como /app
# Todos os comandos seguintes serão executados dentro desse diretório
WORKDIR /app


# Inicia um novo estágio chamado "deps", baseado no estágio "base"
# Esse estágio será responsável por instalar as dependências do projeto
FROM base AS deps

# Copia apenas os arquivos de dependências do projeto
# package.json → lista as dependências
# package-lock.json* → garante instalação exata das versões
# O * permite funcionar mesmo se o arquivo não existir
COPY package.json package-lock.json* ./

# Instala todas as dependências do projeto usando npm
# Isso cria a pasta node_modules dentro do container
RUN npm install


# Inicia um novo estágio chamado "builder"
# Esse estágio será responsável por compilar (buildar) a aplicação
FROM base AS builder

# Copia a pasta node_modules gerada no estágio "deps"
# Isso evita reinstalar as dependências novamente
COPY --from=deps /app/node_modules ./node_modules

# Copia todo o restante do código da aplicação para dentro do container
COPY . .

# Executa o comando de build da aplicação
# Em aplicações Next.js normalmente gera a pasta .next com os arquivos otimizados
RUN npm run build


# Inicia o estágio final chamado "runner"
# Esse estágio será a imagem final que rodará em produção
FROM node:20-alpine AS runner

# Define novamente o diretório de trabalho
WORKDIR /app

# Define variáveis de ambiente
# NODE_ENV=production → ativa otimizações do Node.js para produção
ENV NODE_ENV=production

# Define a porta que a aplicação usará internamente
ENV PORT=3000


# Cria um grupo chamado "nextjs" dentro do container
# -S cria um grupo de sistema (mais seguro e leve)
# Depois cria um usuário "nextjs" pertencente a esse grupo
# Isso evita rodar o container como root (boa prática de segurança)
RUN addgroup -S nextjs && adduser -S nextjs -G nextjs


# Copia os arquivos públicos da aplicação gerados no build
# Exemplo: imagens, favicon, etc
COPY --from=builder /app/public ./public

# Copia a versão standalone do Next.js gerada no build
# Esse modo inclui apenas os arquivos necessários para rodar a aplicação
COPY --from=builder /app/.next/standalone ./

# Copia os arquivos estáticos do Next.js
# Esses arquivos incluem CSS, JS e assets gerados durante o build
COPY --from=builder /app/.next/static ./.next/static


# Define que o container será executado usando o usuário "nextjs"
# Isso aumenta a segurança evitando execução como root
USER nextjs

# Informa ao Docker que a aplicação escuta na porta 3000
# Isso não abre a porta, apenas documenta para o Docker
EXPOSE 3000

# Comando que será executado quando o container iniciar
# Aqui ele executa o servidor Node que inicia a aplicação Next.js
CMD ["node", "server.js"]