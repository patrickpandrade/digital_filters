# Visualização Interativa de Filtros Digitais para Proteção de Sistemas Elétricos

Ferramentas interativas para visualizar o funcionamento dos algoritmos de filtragem digital utilizados em relés de proteção (IEDs) — da Transformada Discreta de Fourier ao Filtro Cosseno.

![DFT Visualization](https://img.shields.io/badge/HTML-standalone-blue) ![License](https://img.shields.io/badge/license-MIT-green) ![Status](https://img.shields.io/badge/status-stable-brightgreen)

---

## Sobre o Projeto

Relés digitais de proteção (IEDs) processam sinais de tensão e corrente amostrados para extrair fasores que alimentam as funções de proteção — distância, diferencial, sobrecorrente direcional, entre outras. O algoritmo que realiza essa extração é, na maioria dos casos, uma variante da Transformada Discreta de Fourier (DFT) ou do Filtro Cosseno.

Estas ferramentas permitem **visualizar passo a passo** como esses algoritmos operam sobre um sinal composto por múltiplas harmônicas, tornando tangíveis conceitos que normalmente ficam abstratos em livros e manuais.

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `dft-viz.html` | Visualização da DFT e do Filtro de Fourier de onda completa |
| `cosine-filter-viz.html` | Visualização do Filtro Cosseno com deslocamento temporal de N/4 |

Ambos são **arquivos HTML autocontidos** — basta abrir no navegador. Carregam React e Babel via CDN (Cloudflare) na primeira execução.

---

## dft-viz.html — DFT e Filtro de Fourier

### O que faz

Demonstra como a DFT extrai a componente de uma frequência específica de um sinal amostrado, através da correlação com senoides de teste (cosseno e seno).

### Funcionalidades

- **Sinal configurável** com 5 componentes selecionáveis: fundamental (1ª), 2ª, 3ª, 5ª e 7ª harmônicas
- **16 amostras por ciclo** (N=16), representativo de relés comerciais
- **Seletor de bin de frequência** (k=0 a 8, bins úteis até Nyquist)
- **Passo a passo visual da DFT** para o bin selecionado:
  - Cosseno de teste cos(2πkn/N)
  - Produto ponto-a-ponto x[n]·cos → parte real
  - Seno de teste −sin(2πkn/N)
  - Produto ponto-a-ponto x[n]·(−sin) → parte imaginária
- **Diagrama fasorial** com projeções Re/Im, magnitude e ângulo
- **Relação DFT ↔ Filtro de Fourier** mostrando a conversão entre a DFT bruta e o estimador de fasor normalizado por 2/N
- **Espectro de magnitude** com bins úteis
- **Resposta em frequência** dinâmica do filtro sintonizado no bin selecionado

### Fórmulas implementadas

**DFT geral:**

```
Re{X[k]} = Σ x[n]·cos(2πkn/N)
Im{X[k]} = −Σ x[n]·sin(2πkn/N)
```

**Filtro de Fourier (estimador de fasor):**

```
V_Real = (2/N)·Σ Sₙ·sin(2πkn/N)
V_Imag = (2/N)·Σ Sₙ·cos(2πkn/N)
```

A troca de seno/cosseno entre as duas formulações reflete a convenção de referência senoidal usada em proteção de sistemas de potência, resultando em uma rotação de 90° do fasor: V = (2/N)·j·X[k].

---

## cosine-filter-viz.html — Filtro Cosseno

### O que faz

Demonstra o Filtro Cosseno de 1 ciclo, que utiliza apenas coeficientes cosseno para extrair o fasor completo. A parte imaginária é obtida aplicando os mesmos coeficientes cosseno ao sinal deslocado de N/4 amostras no tempo.

### Funcionalidades

- **Sinal de 2 ciclos** (32 amostras) com as mesmas 5 harmônicas selecionáveis
- **Visualização das janelas** de amostragem: parte real (n=0…15, verde) e parte imaginária (n=4…19, rosa), com a sobreposição destacada em roxo
- **Passo a passo em dois blocos separados:**
  - **Bloco Real:** amostras do 1º ciclo × cosseno de teste → soma → Re
  - **Bloco Imaginário:** amostras deslocadas de N/4 × mesmo cosseno → soma → −Im
- **Diagrama fasorial** com as fórmulas do filtro cosseno
- **Espectro** mostrando apenas bins úteis (h=0 a N/2)
- **Resposta em frequência** do filtro cosseno
- **Limitação documentada:** o deslocamento temporal de N/4 produz uma defasagem de h×90° — para harmônicas pares (h=2, 4, 6), a defasagem é múltiplo de 180° e o filtro falha na extração

### Fórmulas implementadas

```
X_Re[h] = (2/N)·Σ x[n]·cos(2πhn/N)           — n = 0…N-1
X_Im[h] = −(2/N)·Σ x[n+N/4]·cos(2πhn/N)      — amostras N/4…N/4+N-1
```

### Observação sobre harmônicas pares

Ao selecionar h=2 com a 2ª harmônica ativa, o filtro retorna magnitude zero. Isso ocorre porque o deslocamento de N/4=4 amostras causa uma defasagem de 2×90°=180° na 2ª harmônica — o sinal deslocado é o negativo do original, sem informação em quadratura. Esta é uma limitação conhecida e aceita no contexto de proteção, onde o alvo primário é a fundamental (ímpar).

---

## Como Usar

1. Baixe o arquivo `.html` desejado
2. Abra com duplo-clique em qualquer navegador moderno (Chrome, Firefox, Edge, Safari)
3. Necessita conexão com internet na primeira abertura (para carregar React/Babel via CDN ~140 KB)
4. Após o primeiro carregamento, as bibliotecas ficam em cache

### Interação

- **Botões de harmônicas:** ative/desative componentes do sinal para observar como o espectro e o processamento mudam
- **Seletor k/h:** escolha o bin de frequência e observe o passo a passo do filtro para aquela frequência
- **Resposta em frequência:** compare visualmente o comportamento do filtro em diferentes frequências-alvo

---

## Contexto Técnico

Estas visualizações são complementares e permitem comparar diretamente:

| Característica | Filtro de Fourier (DFT) | Filtro Cosseno |
|---|---|---|
| Bases utilizadas | cos + sin | cos apenas |
| Amostras necessárias | N | N + N/4 |
| Tabelas em memória | 2 (sin + cos) | 1 (cos) |
| Latência | 1 ciclo | 1¼ ciclo |
| Harmônicas pares | Funciona | Não funciona (h×90° ≠ 90°) |
| Rejeição de DC exp. | Boa | Muito boa (duplo diferencial) |
| Implementação recursiva | Sim | Não (apenas não-recursivo) |

### Parâmetros

- **N = 16 amostras/ciclo** (equivalente a 960 Hz de taxa de amostragem a 60 Hz)
- **Resolução em frequência:** cada bin corresponde a 60 Hz (fundamental)
- **Nyquist:** bin k=8 corresponde a 480 Hz

---

## Referências

- SEL-400 Series Relays — Instruction Manual, Section 9: Data Processing. Schweitzer Engineering Laboratories, 2026.
- Noções de Proteção Digital — Filtragem Digital e Algoritmos. Virtus Consultoria e Serviços Ltda., Curso de Proteção, Ed. 5.
- Phadke, A.G. & Thorp, J.S. — *Computer Relaying for Power Systems*. John Wiley & Sons, 2009.
- US Patent 6,154,687A — Modified Cosine Filters. Schweitzer Engineering Laboratories.

---

## Autor

**Patrick Rafael Portes Andrade**
Engenheiro Eletricista · Siemens Brasil · Doutorando em Engenharia Elétrica (UNIFEI)

---

## Licença

MIT License — livre para uso, modificação e distribuição.
