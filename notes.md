docker compose -f docker-compose.dev.yml down -v
docker compose -f docker-compose.dev.yml up -d db redis


npx nx start twenty-server --yes
npx nx start twenty-front --yes

npm run start:front


