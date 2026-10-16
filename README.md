# APM 4.0 — site de monitoramento industrial

## Arquivos
- `index.html`: estrutura do dashboard
- `style.css`: aparência
- `app.js`: interface, gráficos, leitura e diagnóstico
- `config.js`: URL da API e modo demonstração
- `Code.gs`: versão da API Google Apps Script com suporte a GET e POST

## Abrir no VS Code
1. Extraia o ZIP e abra a pasta no VS Code.
2. Abra `index.html` com a extensão Live Server (ou use um servidor local estático).
3. O site abre com dados **simulados**; não são leituras reais.

## Integrar à Planilha Google
1. Abra sua planilha e vá em Extensões > Apps Script.
2. Substitua o código do projeto por `Code.gs`, salve e implante como **Aplicativo da Web**.
3. Configure as permissões de acesso adequadas. **Atenção:** implantação pública expõe os dados retornados pelo GET e pode permitir POST por terceiros. Não use dados industriais sensíveis nessa configuração sem autenticação e um backend protegido.
4. Copie a URL `/exec` e cole em `config.js` em `API_URL`.
5. Altere `DEMO` para `false` e recarregue o site.
6. O POST de sensores/simulador envia JSON à URL `/exec`; o GET fornece as últimas 200 linhas para o site.

## Observações técnicas
- O Apps Script não implementa de forma convencional o preflight `OPTIONS` do navegador. Este site apenas faz GET para evitar POST do browser; POST deve ser realizado por cliente compatível ou backend.
- Dependendo da implantação e dos redirecionamentos do Google, requisições diretas do navegador podem ser bloqueadas por CORS. Nesse caso, use um backend/proxy seguro para consultar a API; não desabilite a segurança do navegador.
- O painel recalcula o diagnóstico a partir das leituras; a classificação armazenada na planilha pode diferir se os limites forem alterados.
- Os limites são demonstrativos e devem ser aprovados por profissionais responsáveis pelo equipamento. O sistema não aciona paradas de máquinas.
- `Índice de saúde` é um valor recebido da fonte, não calculado pelo site.
