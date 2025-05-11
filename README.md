# Diagrama de Bode TC

**Trabalho**

**Prova 3.2 Teoria dos Circuitos**

**Aluno: Pedro Arthur Oliveira dos Santos**

**Professor: Ricardo Pinheiro**

***

**Questão**

Normalizando o único polinômio que não está normalizado, no numerador, temos:

$$ (26 \times 10^{-8}) s^2 (s+205) $$

então a forma final:

*(A fonte não mostra a forma final da função de transferência após a normalização, apenas o polinômio normalizado do numerador)*

É possível ver no numerador uma **frequência de corte 1 rad/s**, de **ordem 2**. E uma frequência de corte de **205 rad/s**, de **ordem 1**.

Já no denominador vemos uma frequência de corte de **segundo grau** de $$ \sqrt{4 \times 10^9} = 63245.55 \text{ rad/s} $$ (calculada a partir do código), e uma de $$ \sqrt{0.4 \times 10^{12}} = 632455.5 \text{ rad/s} $$ (calculada a partir do código).
*(Nota: A fonte original apenas indica as variáveis wcd_1 e wcd_2 sem mostrar seus valores calculados explicitamente no texto, mas o código mostra o cálculo usando `sqrt(4*10^9)` e `sqrt(0.4*10^12)`. O valor de wcd_2 no código é `sqrt(0.4*10^12)`, diferente do valor de `sqrt(4*10^11)` mencionado no texto. Utilizei os valores calculados do código como base).*

***

**Explicação do código Scilab**

O código FINAL Scilab é apresentado nos excertos.

**Definição da Função de Transferência**

```scilab
s = poly(0, 's');
H_s = (26 * 10 ^ (-8) * s ^ 2 * (s + 205)) / ...
((s ^ 2 / (4 * 10 ^ 9) + 1.6 * 10 ^ (-6) * s + 1) * ...
(s ^ 2 / (4 * 10 ^ 11) + 0.16 * 10 ^ (-6) * s + 1))
H = syslin('c',H_s);
```

O trecho de código acima cria uma variável `s` a partir da string “s” que pode ser usada para implementar uma expressão algébrica. A função de transferência da questão é definida usando a variável `H_s`. A função `syslin('c', H_s);` cria o vetor da função de transferência que pode ser usado no plot.

**Plotagem das Curvas de Bode Reais (Magnitude e Fase)**

```scilab
scf(0); // nova janela de gráfico
bode(H, 0.001, 10 ^ 8, "rad");
bode_asymp(H, 0.001, 10 ^ 8)
title('Diagrama de Bode');
xlabel('Frequência (rad/s)');
ylabel('Magnitude (dB)');
xgrid();
```

Este trecho cria uma janela gráfica com as curvas de bode de magnitude e fase, e define os títulos dos eixos e do gráfico. O plot dos gráficos é dado pela função `bode(H, 0.001, 10^8, "rad")`. O primeiro parâmetro `H` é a função de transferência matricial. O segundo e terceiro parâmetros (`0.001` e `10^8`) são os limites do eixo da frequência. O quarto parâmetro (`"rad"`) converte o eixo de Hertz para radianos por segundo.

O trecho de código `bode_asymp(H, 0.001, 10^8)` deveria plotar assíntotas tanto para a magnitude quanto para a fase. No entanto, ele plotou apenas as da fase. Por esse motivo, foi desenvolvida uma lógica para o plot das assíntotas manualmente.

**Plot das Assíntotas de Bode (Magnitude Manual)**

O plot das assíntotas de bode para o módulo pode ser feito usando lógica condicional `if then else`, utilizando as características das frequências de corte de primeiro e segundo grau, estudadas no capítulo 10.

O código abaixo implementa os cálculos das assíntotas:

```scilab
w = logspace(-1, 10, 500);

// Definir constantes
wcn_1 = 1;
wcn_2 = 205;
wcd_1 = sqrt(4*10^9);
wcd_2 = sqrt(0.4*10^12); // Nota: Este valor difere ligeiramente do texto
K = (26*10^(-8))/205;

// Inicializar arrays
magnitude_assintota = zeros(size(w));

// Calcular assíntotas de magnitude
for i = 1:length(w)

    if w(i) < wcn_1 then
        h1 = 40*log10(1);
    else
        h1 = 40*log10(w(i)/wcn_1);
    end

    if w(i) < wcn_2 then
        h2 = 20*log10(1);
    else
        h2 = 20*log10(w(i)/wcn_2);
    end

    if w(i) < wcd_1 then
        h3 = 20*log10(1);
    else
        h3 = 40*log10(w(i)/wcd_1);
    end

    if w(i) < wcd_2 then
        h4 = 20*log10(1);
    else
        h4 = 40*log10(w(i)/wcd_2);
    end

    magnitude_assintota(i) = h1 + h2 - h3 - h4;

end

// Adicionar a contribuição do ganho K
magnitude_assintota = magnitude_assintota + 20*log10(K);
```

Este código implementa o padrão de Bode para plotagem do gráfico de módulo. Primeiro, um eixo logarítmico (`w`) para plotar o gráfico é criado. Depois, as contribuições de cada termo polinomial do numerador e denominador são computadas dentro de um loop. Finalmente, o offset do ganho `K` é adicionado.

**Aproximação e Padrão de Bode**

O cálculo das assíntotas segue o padrão de Bode:

*   **Termo de 2ª Ordem no Numerador** (implicado pelo uso de `40*log10` antes e depois da frequência de corte `wcn_1` nos cálculos de `h1`):
    *   Se $$ \omega < \omega_{cn1} $$, a contribuição é constante (0 dB, ou $$ 40 \log_{10}(1) $$).
    *   Se $$ \omega > \omega_{cn1} $$, a contribuição aumenta com uma inclinação de +40 dB/década ($$ 40 \log_{10}\left(\frac{\omega}{\omega_{cn1}}\right) $$).
*   **Termo de 1ª Ordem no Numerador** (implicado pelo uso de `20*log10` antes e depois da frequência de corte `wcn_2` nos cálculos de `h2`):
    *   Se $$ \omega < \omega_{cn2} $$, a contribuição é constante (0 dB, ou $$ 20 \log_{10}(1) $$).
    *   Se $$ \omega > \omega_{cn2} $$, a contribuição aumenta com uma inclinação de +20 dB/década ($$ 20 \log_{10}\left(\frac{\omega}{\omega_{cn2}}\right) $$).
*   **Termo de 2ª Ordem no Denominador** (implicado pelo uso de `20*log10` antes e `40*log10` depois da frequência de corte `wcd_1` nos cálculos de `h3`, e a subtração de `h3` do total):
    *   Se $$ \omega < \omega_{cd1} $$, a contribuição é constante (0 dB, ou $$ 20 \log_{10}(1) $$).
    *   Se $$ \omega > \omega_{cd1} $$, a contribuição diminui com uma inclinação de -40 dB/década ($$ -40 \log_{10}\left(\frac{\omega}{\omega_{cd1}}\right) $$). *(Nota: O código calcula +40 e depois subtrai)*
*   **Termo de 2ª Ordem no Denominador** (implicado pelo uso de `20*log10` antes e `40*log10` depois da frequência de corte `wcd_2` nos cálculos de `h4`, e a subtração de `h4` do total):
    *   Se $$ \omega < \omega_{cd2} $$, a contribuição é constante (0 dB, ou $$ 20 \log_{10}(1) $$).
    *   Se $$ \omega > \omega_{cd2} $$, a contribuição diminui com uma inclinação de -40 dB/década ($$ -40 \log_{10}\left(\frac{\omega}{\omega_{cd2}}\right) $$). *(Nota: O código calcula +40 e depois subtrai)*

Em cada loop, é computada a contribuição de cada polinômio, considerando o valor de $$ \omega $$ para cada frequência de corte. No final, é adicionado o offset ao gráfico de magnitude, dado por $$ 20 \log_{10}(K) $$.

**Plotagem das Assíntotas de Magnitude Manual**

É feito então o plot dos gráficos com este trecho de código, mostrando as assíntotas da magnitude da função de transferência em função do eixo logarítmico $$ \omega $$:

```scilab
scf(1); //Nova janela
semilogx(w, magnitude_assintota, "b-");
title('Curva de Bode - Magnitude com Assíntotas');
xlabel('Frequência (rad/s)');
ylabel('Magnitude (dB)');
xgrid();
```

***

**Resultado dos Gráficos**

**Módulo de Yeq(S) (Curva Real e Assíntotas)**
*(Imagem fornecida na fonte mostrando a curva de magnitude real e a curva assintótica calculada manualmente)*

**Fase de Yeq(S) (Curva Real e Assíntotas)**
*(Imagem fornecida na fonte mostrando a curva de fase real e a curva assintótica (apenas fase) plotada pela função `bode_asymp`)*

**Gráfico das Curvas de Bode sem assíntotas**
*(Imagem fornecida na fonte mostrando as curvas de magnitude e fase reais plotadas usando apenas `bode(H, 0.001, 10^8, "rad")`)*

```scilab
usando apenas
bode(H, 0.001, 10^8, "rad");
```
