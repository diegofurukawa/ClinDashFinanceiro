# Documentação: Dashboard Financeiro com ASP.NET Core e Vue.js

## 1. Visão Geral do Projeto

Este projeto consiste em um dashboard financeiro desenvolvido com ASP.NET Core para o backend e Vue.js para o frontend. O sistema permitirá visualizar métricas financeiras como faturamento, recebimento, lucro, análise por categoria e acompanhamento de metas.

### 1.1. Objetivos

- Criar uma API RESTful em ASP.NET Core para disponibilizar os dados financeiros
- Desenvolver um frontend responsivo em Vue.js com visualizações interativas
- Implementar gráficos e componentes visuais usando ECharts
- Garantir desempenho e segurança na comunicação entre frontend e backend

### 1.2. Tecnologias Principais

| Backend | Frontend | DevOps |
|---------|----------|--------|
| ASP.NET Core 8.0 | Vue.js 3 | Git |
| Entity Framework Core | ECharts/Vue-ECharts | Docker |
| SQL Server/PostgreSQL | Vue Router | CI/CD (Azure DevOps/GitHub Actions) |
| Identity (autenticação) | Pinia (gerenciamento de estado) | Testes Automatizados |

## 2. Arquitetura do Sistema

### 2.1. Arquitetura de Alto Nível

```

### 7.2. Otimizações para Vue.js

1. Lazy loading de componentes e rotas:
   ```javascript
   // router/index.js
   const routes = [
     {
       path: '/',
       name: 'Dashboard',
       component: () => import('../views/DashboardView.vue') // Lazy loading
     },
     {
       path: '/relatorios',
       name: 'Relatorios',
       component: () => import('../views/RelatoriosView.vue')
     }
   ]
   ```

2. Virtualização de listas longas:
   ```vue
   <template>
     <RecycleScroller
       class="scroller"
       :items="categorias"
       :item-size="40"
       key-field="id"
       v-slot="{ item }"
     >
       <div class="categoria-item">
         <span>{{ item.nome }}</span>
         <span>{{ formatarMoeda(item.valor) }}</span>
       </div>
     </RecycleScroller>
   </template>
   
   <script setup>
   import { RecycleScroller } from 'vue-virtual-scroller'
   import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'
   </script>
   ```

3. Utilização de componentes assíncronos em ECharts para melhorar o tempo de carregamento:
   ```javascript
   // Carregamento assíncrono dos componentes do ECharts
   import { use } from 'echarts/core';
   
   // Carregar apenas os componentes necessários
   use([
     CanvasRenderer,
     BarChart,
     LineChart,
     GridComponent,
     TooltipComponent
   ]);
   ```

4. Implementação de PWA (Progressive Web App) para uso offline:
   ```bash
   # Instalar o plugin PWA
   npm install -D @vite/plugin-pwa
   ```

   ```javascript
   // vite.config.js
   import { defineConfig } from 'vite'
   import vue from '@vitejs/plugin-vue'
   import { VitePWA } from 'vite-plugin-pwa'

   export default defineConfig({
     plugins: [
       vue(),
       VitePWA({
         registerType: 'autoUpdate',
         includeAssets: ['favicon.ico', 'robots.txt', 'apple-touch-icon.png'],
         manifest: {
           name: 'Dashboard Financeiro',
           short_name: 'DashFin',
           description: 'Dashboard para visualização de dados financeiros',
           theme_color: '#304592',
           icons: [
             // Configurações de ícones...
           ]
         },
         workbox: {
           // Estratégias de cache
           runtimeCaching: [
             {
               urlPattern: ({ url }) => url.pathname.startsWith('/api/dashboard/kpis'),
               handler: 'StaleWhileRevalidate',
               options: {
                 cacheName: 'api-cache',
                 expiration: {
                   maxEntries: 10,
                   maxAgeSeconds: 60 * 60 // 1 hora
                 }
               }
             }
           ]
         }
       })
     ]
   })
   ```

## 8. Testes Automatizados

### 8.1. Testes de Backend (ASP.NET Core)

```csharp
// DashboardFinanceiro.API.Tests/DashboardControllerTests.cs
using System.Threading.Tasks;
using DashboardFinanceiro.API.Controllers;
using DashboardFinanceiro.API.Services;
using Microsoft.AspNetCore.Mvc;
using Moq;
using Xunit;

public class DashboardControllerTests
{
    private readonly Mock<IDashboardService> _mockService;
    private readonly DashboardController _controller;

    public DashboardControllerTests()
    {
        _mockService = new Mock<IDashboardService>();
        _controller = new DashboardController(_mockService.Object);
    }

    [Fact]
    public async Task GetKPIs_ReturnsOkResult()
    {
        // Arrange
        int ano = 2024;
        int mes = 4;
        var kpiViewModel = new KPIViewModel
        {
            Faturamento = 2343622.80m,
            Recebimento = 2163553.68m,
            Lucro = 584101.62m,
            MetaFaturamento = 2398500.00m,
            TicketMedio = 1093.62m
        };
        
        _mockService.Setup(s => s.GetKPIsAsync(ano, mes))
            .ReturnsAsync(kpiViewModel);

        // Act
        var result = await _controller.GetKPIs(ano, mes);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnValue = Assert.IsType<KPIViewModel>(okResult.Value);
        Assert.Equal(kpiViewModel.Faturamento, returnValue.Faturamento);
        Assert.Equal(kpiViewModel.Recebimento, returnValue.Recebimento);
        Assert.Equal(kpiViewModel.Lucro, returnValue.Lucro);
    }

    [Fact]
    public async Task GetFaturamentoMensal_ReturnsCorrectData()
    {
        // Arrange
        int ano = 2024;
        var faturamentoViewModel = new FaturamentoMensalViewModel
        {
            DadosAnoAtual = new decimal[] { 190000, 210000, 205000, 220000, 280000, 200000, 210000, 180000, 220000, 240000, 270000, 240000 },
            DadosAnoAnterior = new decimal[] { 170000, 180000, 190000, 200000, 220000, 195000, 200000, 170000, 210000, 230000, 210000, 210000 },
            Metas = new decimal[] { 200000, 210000, 215000, 225000, 235000, 215000, 225000, 205000, 230000, 240000, 250000, 245000 }
        };
        
        _mockService.Setup(s => s.GetFaturamentoMensalAsync(ano))
            .ReturnsAsync(faturamentoViewModel);

        // Act
        var result = await _controller.GetFaturamentoMensal(ano);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnValue = Assert.IsType<FaturamentoMensalViewModel>(okResult.Value);
        Assert.Equal(faturamentoViewModel.DadosAnoAtual.Length, returnValue.DadosAnoAtual.Length);
        Assert.Equal(faturamentoViewModel.DadosAnoAnterior.Length, returnValue.DadosAnoAnterior.Length);
        Assert.Equal(faturamentoViewModel.Metas.Length, returnValue.Metas.Length);
    }
}
```

### 8.2. Testes de Frontend (Vue.js)

```javascript
// DashboardFinanceiro.Web.Tests/components/KpiCard.spec.js
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import KpiCard from '../../src/components/dashboard/KpiCard.vue'

describe('KpiCard.vue', () => {
  it('renderiza o título e valor corretamente', () => {
    const wrapper = mount(KpiCard, {
      props: {
        title: 'FATURAMENTO',
        value: 'R$ 2.343.622,80',
        color: 'blue',
        chartData: [120, 200, 150, 80, 70, 110, 130]
      }
    })
    
    expect(wrapper.find('.card-title').text()).toBe('FATURAMENTO')
    expect(wrapper.find('.card-value').text()).toBe('R$ 2.343.622,80')
  })
  
  it('aplica a classe de cor correta com base na propriedade color', () => {
    const wrapper = mount(KpiCard, {
      props: {
        title: 'LUCRO',
        value: 'R$ 584.101,62',
        color: 'purple',
        chartData: [30, 40, 35, 50, 59, 70, 80]
      }
    })
    
    expect(wrapper.classes()).toContain('lucro-card')
  })
})
```

```javascript
// DashboardFinanceiro.Web.Tests/store/dashboard.spec.js
import { setActivePinia, createPinia } from 'pinia'
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { useDashboardStore } from '../../src/store/dashboard'
import api from '../../src/services/api'

// Mock do módulo api
vi.mock('../../src/services/api', () => ({
  default: {
    getKPIs: vi.fn(),
    getFaturamentoMensal: vi.fn(),
    getCategoriasAnalise: vi.fn(),
    getMetasRealizacao: vi.fn()
  }
}))

describe('Dashboard Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('busca e armazena os dados do dashboard corretamente', async () => {
    // Arrange
    const store = useDashboardStore()
    
    const mockKpisResponse = {
      data: {
        faturamento: 2343622.80,
        recebimento: 2163553.68,
        lucro: 584101.62,
        periodoAtual: {
          valor: 277571.95,
          percentualCrescimento: 10.56
        }
      }
    }
    
    const mockFaturamentoResponse = {
      data: {
        dadosAnoAtual: [190000, 210000, 205000],
        dadosAnoAnterior: [170000, 180000, 190000],
        metas: [200000, 210000, 215000]
      }
    }
    
    const mockCategoriasResponse = {
      data: [
        { nome: 'Categoria 1', valor: 1807005.60 },
        { nome: 'Categoria 2', valor: 151055.65 }
      ]
    }
    
    const mockMetasResponse = {
      data: {
        percentualMetaAlcancada: 97.71,
        realizado: 277571.95,
        metaAtual: 180180.00
      }
    }
    
    // Configurar mocks
    api.getKPIs.mockResolvedValue(mockKpisResponse)
    api.getFaturamentoMensal.mockResolvedValue(mockFaturamentoResponse)
    api.getCategoriasAnalise.mockResolvedValue(mockCategoriasResponse)
    api.getMetasRealizacao.mockResolvedValue(mockMetasResponse)
    
    // Act
    await store.fetchDashboardData()
    
    // Assert
    expect(store.kpis).toEqual(mockKpisResponse.data)
    expect(store.faturamentoMensal).toEqual(mockFaturamentoResponse.data)
    expect(store.categorias).toEqual(mockCategoriasResponse.data)
    expect(store.metasRealizacao).toEqual(mockMetasResponse.data)
    expect(store.periodoAtual).toEqual(mockKpisResponse.data.periodoAtual)
  })
  
  it('atualiza os filtros e busca novos dados', async () => {
    // Arrange
    const store = useDashboardStore()
    const newAno = 2023
    const newMes = 6
    
    api.getKPIs.mockResolvedValue({ data: {} })
    api.getFaturamentoMensal.mockResolvedValue({ data: {} })
    api.getCategoriasAnalise.mockResolvedValue({ data: [] })
    api.getMetasRealizacao.mockResolvedValue({ data: {} })
    
    // Act
    await store.setFiltros(newAno, newMes)
    
    // Assert
    expect(store.anoSelecionado).toBe(newAno)
    expect(store.mesSelecionado).toBe(newMes)
    expect(api.getKPIs).toHaveBeenCalledWith(newAno, newMes)
    expect(api.getFaturamentoMensal).toHaveBeenCalledWith(newAno)
    expect(api.getCategoriasAnalise).toHaveBeenCalledWith(newAno, newMes)
    expect(api.getMetasRealizacao).toHaveBeenCalledWith(newAno, newMes)
  })
})
```

## 9. CI/CD com GitHub Actions

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test-api:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 8.0.x
        
    - name: Restore dependencies
      run: dotnet restore ./src/DashboardFinanceiro.API/DashboardFinanceiro.API.csproj
      
    - name: Build
      run: dotnet build ./src/DashboardFinanceiro.API/DashboardFinanceiro.API.csproj --no-restore
      
    - name: Test
      run: dotnet test ./tests/DashboardFinanceiro.API.Tests/DashboardFinanceiro.API.Tests.csproj --no-build --verbosity normal

  build-and-test-web:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: |
        cd src/DashboardFinanceiro.Web
        npm ci
        
    - name: Build
      run: |
        cd src/DashboardFinanceiro.Web
        npm run build
        
    - name: Test
      run: |
        cd src/DashboardFinanceiro.Web
        npm run test:unit
        
  deploy:
    needs: [build-and-test-api, build-and-test-web]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    # Deploy para Azure App Service
    - name: Deploy API to Azure
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'dashboard-financeiro-api'
        publish-profile: ${{ secrets.AZURE_API_PUBLISH_PROFILE }}
        package: './src/DashboardFinanceiro.API/bin/Release/net8.0/publish'
        
    - name: Deploy Web to Azure
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'dashboard-financeiro-web'
        publish-profile: ${{ secrets.AZURE_WEB_PUBLISH_PROFILE }}
        package: './src/DashboardFinanceiro.Web/dist'
```

## 10. Plano de Implementação

### 10.1. Cronograma Sugerido

| Fase | Descrição | Duração Estimada |
|------|-----------|------------------|
| **1. Análise e Planejamento** | Definição detalhada de requisitos, modelagem de dados | 1 semana |
| **2. Setup do Projeto** | Configuração dos ambientes de backend e frontend | 2-3 dias |
| **3. Desenvolvimento Backend** | Implementação dos models, controllers e serviços ASP.NET Core | 2 semanas |
| **4. Desenvolvimento Frontend** | Implementação dos componentes Vue.js e integração com ECharts | 2 semanas |
| **5. Integração e Testes** | Integração entre frontend e backend, testes automatizados | 1 semana |
| **6. Otimização e Refinamento** | Melhorias de desempenho, UX e refinamentos visuais | 1 semana |
| **7. Implantação** | Setup de ambiente de produção e implantação | 2-3 dias |
| **8. Documentação Final** | Finalização da documentação técnica e de usuário | 2-3 dias |

### 10.2. Próximos Passos

1. **Imediato (Semana 1):**
   - Definir a estrutura de dados detalhada
   - Criar o projeto base ASP.NET Core
   - Configurar o projeto Vue.js
   - Configurar o controle de versão Git

2. **Curto Prazo (Semanas 2-4):**
   - Desenvolver as APIs backend essenciais
   - Implementar os componentes principais do dashboard
   - Estabelecer a autenticação e segurança
   - Criar os primeiros gráficos com ECharts

3. **Médio Prazo (Semanas 5-6):**
   - Implementar recursos avançados (filtros, comparativos)
   - Refinar a experiência do usuário
   - Executar testes de desempenho e otimização
   - Preparar ambiente de produção

## 11. Recursos Adicionais

### 11.1. Documentação Oficial

- [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core)
- [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core)
- [Vue.js](https://vuejs.org/guide/introduction.html)
- [ECharts](https://echarts.apache.org/en/index.html)
- [Vue ECharts](https://github.com/ecomfe/vue-echarts)

### 11.2. Tutoriais Recomendados

- [Criando uma API RESTful com ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api)
- [Introdução ao Vue.js 3](https://vuejs.org/tutorial)
- [Primeiros passos com ECharts](https://echarts.apache.org/en/tutorial.html)
- [Autenticação JWT em ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/authentication)

### 11.3. Ferramentas Úteis

- [Visual Studio 2022](https://visualstudio.microsoft.com/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Vue DevTools](https://devtools.vuejs.org/)
- [Postman](https://www.postman.com/) (para testar APIs)
- [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) (para CI/CD)
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│                 │      │                 │      │                 │
│  Cliente        │◄────►│  API ASP.NET    │◄────►│  Banco de       │
│  (Vue.js SPA)   │      │  Core           │      │  Dados          │
│                 │      │                 │      │                 │
└─────────────────┘      └─────────────────┘      └─────────────────┘
```

### 2.2. Estrutura de Diretórios

```
DashboardFinanceiro/
├── src/
│   ├── DashboardFinanceiro.API/         # Projeto ASP.NET Core API
│   │   ├── Controllers/                 # Controllers da API
│   │   ├── Models/                      # Domain models
│   │   ├── Data/                        # Contexto EF e repositórios
│   │   ├── Services/                    # Serviços de negócio
│   │   └── Program.cs                   # Ponto de entrada da aplicação
│   │
│   └── DashboardFinanceiro.Web/         # Frontend Vue.js
│       ├── public/                      # Arquivos estáticos
│       └── src/
│           ├── assets/                  # Imagens e recursos
│           ├── components/              # Componentes Vue
│           │   ├── dashboard/           # Componentes do dashboard
│           │   └── shared/              # Componentes compartilhados
│           ├── services/                # Serviços de API
│           ├── store/                   # Estado da aplicação (Pinia)
│           ├── views/                   # Páginas principais
│           ├── App.vue                  # Componente raiz
│           └── main.js                  # Ponto de entrada Vue
│
├── tests/                               # Testes automatizados
│   ├── DashboardFinanceiro.API.Tests/   # Testes de backend
│   └── DashboardFinanceiro.Web.Tests/   # Testes de frontend
│
├── .gitignore
├── README.md
└── docker-compose.yml
```

## 3. Backend: ASP.NET Core API

### 3.1. Configuração Inicial

1. Criar um novo projeto ASP.NET Core Web API:
   ```bash
   dotnet new webapi -n DashboardFinanceiro.API
   ```

2. Adicionar os pacotes NuGet necessários:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.Design
   dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
   ```

### 3.2. Modelo de Dados

Principais entidades do sistema:

```csharp
// Models/RegistroFinanceiro.cs
public class RegistroFinanceiro
{
    public int Id { get; set; }
    public DateTime Data { get; set; }
    public decimal Valor { get; set; }
    public string Categoria { get; set; }
    public TipoRegistro Tipo { get; set; }
    public int MesReferencia { get; set; }
    public int AnoReferencia { get; set; }
}

public enum TipoRegistro
{
    Faturamento,
    Recebimento,
    Despesa
}

// Models/Meta.cs
public class Meta
{
    public int Id { get; set; }
    public string Descricao { get; set; }
    public decimal ValorMeta { get; set; }
    public int MesReferencia { get; set; }
    public int AnoReferencia { get; set; }
    public TipoMeta Tipo { get; set; }
}

public enum TipoMeta
{
    Faturamento,
    Recebimento,
    Lucro
}
```

### 3.3. Controladores API

Principais endpoints:

```csharp
// Controllers/DashboardController.cs
[ApiController]
[Route("api/[controller]")]
public class DashboardController : ControllerBase
{
    private readonly IDashboardService _dashboardService;

    public DashboardController(IDashboardService dashboardService)
    {
        _dashboardService = dashboardService;
    }

    [HttpGet("kpis")]
    public async Task<IActionResult> GetKPIs([FromQuery] int ano, [FromQuery] int mes)
    {
        var kpis = await _dashboardService.GetKPIsAsync(ano, mes);
        return Ok(kpis);
    }

    [HttpGet("faturamento-mensal")]
    public async Task<IActionResult> GetFaturamentoMensal([FromQuery] int ano)
    {
        var dados = await _dashboardService.GetFaturamentoMensalAsync(ano);
        return Ok(dados);
    }

    [HttpGet("categorias")]
    public async Task<IActionResult> GetCategoriasAnalise([FromQuery] int ano, [FromQuery] int mes)
    {
        var categorias = await _dashboardService.GetCategoriasAnaliseAsync(ano, mes);
        return Ok(categorias);
    }

    [HttpGet("metas-realizacao")]
    public async Task<IActionResult> GetMetasRealizacao([FromQuery] int ano, [FromQuery] int mes)
    {
        var metasRealizacao = await _dashboardService.GetMetasRealizacaoAsync(ano, mes);
        return Ok(metasRealizacao);
    }
}
```

### 3.4. Serviços de Negócio

```csharp
// Services/IDashboardService.cs
public interface IDashboardService
{
    Task<KPIViewModel> GetKPIsAsync(int ano, int mes);
    Task<FaturamentoMensalViewModel> GetFaturamentoMensalAsync(int ano);
    Task<List<CategoriaAnaliseViewModel>> GetCategoriasAnaliseAsync(int ano, int mes);
    Task<MetasRealizacaoViewModel> GetMetasRealizacaoAsync(int ano, int mes);
}

// Services/DashboardService.cs
public class DashboardService : IDashboardService
{
    private readonly ApplicationDbContext _context;

    public DashboardService(ApplicationDbContext context)
    {
        _context = context;
    }

    // Implementações dos métodos...
}
```

### 3.5. Configuração do CORS

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("VueAppPolicy", policy =>
    {
        policy.WithOrigins("http://localhost:5173") // URL do frontend Vue
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// ...

app.UseCors("VueAppPolicy");
```

## 4. Frontend: Vue.js com ECharts

### 4.1. Configuração Inicial

1. Criar um novo projeto Vue.js:
   ```bash
   npm init vue@latest DashboardFinanceiro.Web
   cd DashboardFinanceiro.Web
   npm install
   ```

2. Instalar as dependências necessárias:
   ```bash
   npm install axios echarts vue-echarts pinia
   ```

### 4.2. Estrutura de Componentes

#### 4.2.1. Componentes de Dashboard

```
components/dashboard/
├── KpiCard.vue                  # Cartão de indicador principal
├── FaturamentoChart.vue         # Gráfico mensal de faturamento
├── MetaGauge.vue                # Gráfico gauge para metas
├── ComparativoPeriodo.vue       # Comparativo entre períodos 
├── AnaliseCategoria.vue         # Lista de categorias com barras
├── TicketMedio.vue              # Exibição do ticket médio
└── DashboardContainer.vue       # Container principal
```

#### 4.2.2. Serviços API

```javascript
// services/api.js
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://localhost:7123/api',  // URL da sua API ASP.NET Core
  headers: {
    'Content-Type': 'application/json'
  }
});

export default {
  getKPIs(ano, mes) {
    return apiClient.get(`/dashboard/kpis?ano=${ano}&mes=${mes}`);
  },
  getFaturamentoMensal(ano) {
    return apiClient.get(`/dashboard/faturamento-mensal?ano=${ano}`);
  },
  getCategoriasAnalise(ano, mes) {
    return apiClient.get(`/dashboard/categorias?ano=${ano}&mes=${mes}`);
  },
  getMetasRealizacao(ano, mes) {
    return apiClient.get(`/dashboard/metas-realizacao?ano=${ano}&mes=${mes}`);
  }
};
```

#### 4.2.3. Store com Pinia

```javascript
// store/dashboard.js
import { defineStore } from 'pinia';
import api from '@/services/api';

export const useDashboardStore = defineStore('dashboard', {
  state: () => ({
    kpis: {
      faturamento: 0,
      recebimento: 0,
      lucro: 0
    },
    faturamentoMensal: {
      dadosAnoAtual: [],
      dadosAnoAnterior: [],
      metas: []
    },
    categorias: [],
    metasRealizacao: {
      percentualMetaAlcancada: 0,
      realizado: 0,
      metaAtual: 0
    },
    periodoAtual: {
      valor: 0,
      percentualCrescimento: 0
    },
    anoSelecionado: new Date().getFullYear(),
    mesSelecionado: new Date().getMonth() + 1
  }),
  
  actions: {
    async fetchDashboardData() {
      try {
        const [kpisResponse, faturamentoResponse, categoriasResponse, metasResponse] = await Promise.all([
          api.getKPIs(this.anoSelecionado, this.mesSelecionado),
          api.getFaturamentoMensal(this.anoSelecionado),
          api.getCategoriasAnalise(this.anoSelecionado, this.mesSelecionado),
          api.getMetasRealizacao(this.anoSelecionado, this.mesSelecionado)
        ]);
        
        this.kpis = kpisResponse.data;
        this.faturamentoMensal = faturamentoResponse.data;
        this.categorias = categoriasResponse.data;
        this.metasRealizacao = metasResponse.data;
        this.periodoAtual = kpisResponse.data.periodoAtual;
      } catch (error) {
        console.error('Erro ao carregar dados do dashboard:', error);
      }
    },
    
    setFiltros(ano, mes) {
      this.anoSelecionado = ano;
      this.mesSelecionado = mes;
      this.fetchDashboardData();
    }
  }
});
```

#### 4.2.4. Componente Principal do Dashboard

```vue
<!-- views/DashboardView.vue -->
<template>
  <div class="dashboard-container">
    <div class="dashboard-header">
      <h1>Dashboard Financeiro</h1>
      <div class="dashboard-filters">
        <select v-model="anoSelecionado" @change="atualizarDados">
          <option v-for="ano in anos" :key="ano" :value="ano">{{ ano }}</option>
        </select>
        <select v-model="mesSelecionado" @change="atualizarDados">
          <option v-for="(mes, index) in meses" :key="index" :value="index + 1">{{ mes }}</option>
        </select>
      </div>
    </div>
    
    <div class="dashboard-kpis">
      <KpiCard 
        title="FATURAMENTO" 
        :value="formatarMoeda(kpis.faturamento)" 
        color="blue"
        :chart-data="kpis.faturamentoTendencia" 
      />
      <KpiCard 
        title="RECEBIMENTO" 
        :value="formatarMoeda(kpis.recebimento)" 
        color="green"
        :chart-data="kpis.recebimentoTendencia" 
      />
      <KpiCard 
        title="LUCRO" 
        :value="formatarMoeda(kpis.lucro)" 
        color="purple"
        :chart-data="kpis.lucroTendencia" 
      />
    </div>
    
    <div class="dashboard-charts">
      <FaturamentoChart 
        :dados-ano-atual="faturamentoMensal.dadosAnoAtual"
        :dados-ano-anterior="faturamentoMensal.dadosAnoAnterior"
        :dados-meta="faturamentoMensal.metas"
        :meta-anual="kpis.metaFaturamento"
      />
      
      <div class="dashboard-bottom">
        <ComparativoPeriodo 
          :periodo-atual="periodoAtual.valor"
          :periodo-anterior="periodoAtual.valorAnterior"
          :percentual-crescimento="periodoAtual.percentualCrescimento"
        />
        
        <div class="meta-realizacao">
          <div class="valores">
            <div class="realizado">
              <h3>Realizado</h3>
              <div class="valor">{{ formatarMoeda(metasRealizacao.realizado) }}</div>
            </div>
            <div class="meta">
              <h3>Meta</h3>
              <div class="valor">{{ formatarMoeda(metasRealizacao.metaAtual) }}</div>
            </div>
          </div>
          <div class="percentual">
            {{ metasRealizacao.percentualMeta }}%
          </div>
        </div>
        
        <MetaGauge :percentual="metasRealizacao.percentualMetaAlcancada" />
      </div>
      
      <AnaliseCategoria :categorias="categorias" />
      
      <TicketMedio :valor="kpis.ticketMedio" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import { useDashboardStore } from '@/store/dashboard';
import KpiCard from '@/components/dashboard/KpiCard.vue';
import FaturamentoChart from '@/components/dashboard/FaturamentoChart.vue';
import ComparativoPeriodo from '@/components/dashboard/ComparativoPeriodo.vue';
import MetaGauge from '@/components/dashboard/MetaGauge.vue';
import AnaliseCategoria from '@/components/dashboard/AnaliseCategoria.vue';
import TicketMedio from '@/components/dashboard/TicketMedio.vue';

const dashboardStore = useDashboardStore();
const anoSelecionado = ref(new Date().getFullYear());
const mesSelecionado = ref(new Date().getMonth() + 1);
const meses = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
const anos = [2023, 2024, 2025];

// Computed properties para acesso aos dados da store
const kpis = computed(() => dashboardStore.kpis);
const faturamentoMensal = computed(() => dashboardStore.faturamentoMensal);
const categorias = computed(() => dashboardStore.categorias);
const metasRealizacao = computed(() => dashboardStore.metasRealizacao);
const periodoAtual = computed(() => dashboardStore.periodoAtual);

onMounted(() => {
  dashboardStore.fetchDashboardData();
});

function atualizarDados() {
  dashboardStore.setFiltros(anoSelecionado.value, mesSelecionado.value);
}

function formatarMoeda(valor) {
  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL'
  }).format(valor);
}
</script>

<style scoped>
.dashboard-container {
  max-width: 1440px;
  margin: 0 auto;
  padding: 20px;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.dashboard-kpis {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.dashboard-charts {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.dashboard-bottom {
  display: contents;
}

@media (max-width: 1024px) {
  .dashboard-kpis,
  .dashboard-charts {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .dashboard-kpis,
  .dashboard-charts {
    grid-template-columns: 1fr;
  }
}
</style>
```

### 4.3. Gráficos com ECharts

Implementação do gráfico de barras com ECharts:

```vue
<!-- components/dashboard/FaturamentoChart.vue -->
<template>
  <div class="chart-container">
    <div class="chart-header">
      <div class="legenda">
        <span class="item atual">Ano Atual</span>
        <span class="item anterior">Ano Anterior</span>
        <span class="item meta">Meta</span>
      </div>
      <div class="meta-anual">
        Meta Anual: {{ formatarMoeda(metaAnual) }}
      </div>
    </div>
    <v-chart class="chart" :option="chartOption" autoresize />
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { BarChart, LineChart } from 'echarts/charts';
import {
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
} from 'echarts/components';
import VChart from 'vue-echarts';

// Registrar componentes do ECharts
use([
  CanvasRenderer,
  BarChart,
  LineChart,
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
]);

const props = defineProps({
  dadosAnoAtual: {
    type: Array,
    default: () => []
  },
  dadosAnoAnterior: {
    type: Array,
    default: () => []
  },
  dadosMeta: {
    type: Array,
    default: () => []
  },
  metaAnual: {
    type: Number,
    default: 0
  }
});

const meses = ['jan', 'fev', 'mar', 'abr', 'mai', 'jun', 'jul', 'ago', 'set', 'out', 'nov', 'dez'];

const chartOption = computed(() => ({
  tooltip: {
    trigger: 'axis',
    formatter: function(params) {
      let result = `${params[0].name}<br/>`;
      params.forEach(param => {
        const colorSpan = `<span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${param.color};"></span>`;
        const value = new Intl.NumberFormat('pt-BR', {
          style: 'currency',
          currency: 'BRL'
        }).format(param.value);
        result += `${colorSpan}${param.seriesName}: ${value}<br/>`;
      });
      return result;
    }
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  xAxis: {
    type: 'category',
    data: meses,
    axisLine: {
      lineStyle: {
        color: '#999'
      }
    }
  },
  yAxis: {
    type: 'value',
    axisLine: {
      show: false
    },
    axisTick: {
      show: false
    },
    axisLabel: {
      formatter: function(value) {
        return value >= 1000 ? (value / 1000) + 'k' : value;
      }
    },
    splitLine: {
      lineStyle: {
        type: 'dashed',
        color: '#eee'
      }
    }
  },
  series: [
    {
      name: 'Ano Atual',
      type: 'bar',
      data: props.dadosAnoAtual,
      barWidth: '20%',
      itemStyle: {
        color: '#304592'
      }
    },
    {
      name: 'Ano Anterior',
      type: 'bar',
      data: props.dadosAnoAnterior,
      barWidth: '20%',
      itemStyle: {
        color: '#e0e0e0'
      }
    },
    {
      name: 'Meta',
      type: 'line',
      data: props.dadosMeta,
      symbolSize: 6,
      symbol: 'circle',
      smooth: true,
      itemStyle: {
        color: '#00b894'
      },
      lineStyle: {
        width: 3,
        color: '#00b894'
      }
    }
  ]
}));

function formatarMoeda(valor) {
  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL'
  }).format(valor);
}
</script>

<style scoped>
.chart-container {
  background-color: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  grid-column: span 3;
}

.chart {
  height: 300px;
  width: 100%;
}

.chart-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  align-items: center;
}

.legenda {
  display: flex;
  gap: 15px;
}

.legenda .item {
  display: flex;
  align-items: center;
  font-size: 14px;
}

.legenda .item::before {
  content: "";
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 5px;
}

.legenda .atual::before {
  background-color: #304592;
}

.legenda .anterior::before {
  background-color: #e0e0e0;
}

.legenda .meta::before {
  background-color: #00b894;
}

.meta-anual {
  font-size: 14px;
  font-weight: 500;
}
</style>
```

## 5. Integração e Implantação

### 5.1. Configuração com Docker

```yaml
# docker-compose.yml
version: '3.8'

services:
  sql-server:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrongPassword123!
    ports:
      - "1433:1433"
    volumes:
      - sql-data:/var/opt/mssql

  api:
    build:
      context: ./src/DashboardFinanceiro.API
      dockerfile: Dockerfile
    ports:
      - "7123:80"
    depends_on:
      - sql-server
    environment:
      - ConnectionStrings__DefaultConnection=Server=sql-server;Database=DashboardFinanceiro;User=sa;Password=YourStrongPassword123!;TrustServerCertificate=True
      - ASPNETCORE_ENVIRONMENT=Development

  web:
    build:
      context: ./src/DashboardFinanceiro.Web
      dockerfile: Dockerfile
    ports:
      - "5173:80"
    depends_on:
      - api

volumes:
  sql-data:
```

### 5.2. Configurações para Desenvolvimento

Dockerfile para a API ASP.NET Core:

```dockerfile
# src/DashboardFinanceiro.API/Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["DashboardFinanceiro.API.csproj", "./"]
RUN dotnet restore "DashboardFinanceiro.API.csproj"
COPY . .
RUN dotnet build "DashboardFinanceiro.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "DashboardFinanceiro.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DashboardFinanceiro.API.dll"]
```

Dockerfile para a aplicação Vue.js:

```dockerfile
# src/DashboardFinanceiro.Web/Dockerfile
FROM node:18 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Configuração do Nginx:

```
# src/DashboardFinanceiro.Web/nginx.conf
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # Configuração de proxy para a API
    location /api/ {
        proxy_pass http://api:80/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 6. Segurança e Autenticação

### 6.1. Configuração JWT no Backend

```csharp
// Program.cs
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidAudience = builder.Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
    };
});

builder.Services.AddAuthorization();
```

### 6.2. Autenticação no Frontend Vue

```javascript
// services/auth.js
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://localhost:7123/api',
  headers: {
    'Content-Type': 'application/json'
  }
});

export default {
  login(credentials) {
    return apiClient.post('/auth/login', credentials);
  },
  setAuthToken(token) {
    localStorage.setItem('token', token);
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  },
  clearAuthToken() {
    localStorage.removeItem('token');
    delete axios.defaults.headers.common['Authorization'];
  },
  getAuthToken() {
    return localStorage.getItem('token');
  },
  init() {
    const token = this.getAuthToken();
    if (token) {
      axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    }
  }
};
```

## 7. Performance e Otimização

### 7.1. Dicas de Performance para ASP.NET Core

1. Utilize cache para dados que não mudam frequentemente:
   ```csharp
   builder.Services.AddResponseCaching();
   
   // Em controllers específicos
   [ResponseCache(Duration = 60)] // Cache por 60 segundos