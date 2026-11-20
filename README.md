# StockGain
 Script em Python que consulta o preço histórico de qualquer ação via yfinance e gera dois gráficos: o lucro em reais por ação e a valorização percentual em relação ao preço de compra informado pelo usuário. Ideal para acompanhar o desempenho real de um investimento ao longo do tempo.
"""
Gráfico de Lucro por Ação e Valorização Percentual
Fonte de dados: yfinance
"""

import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import sys

# ============================================================
# COLETA DE DADOS
# ============================================================

def obter_dados(ticker, periodo):
    """Baixa os preços de fechamento ajustados."""
    acao = yf.Ticker(ticker)
    hist = acao.history(period=periodo)
    if hist.empty:
        raise ValueError(f"Nenhum dado retornado para {ticker}")
    return hist[['Close']].copy()

# ============================================================
# CÁLCULOS
# ============================================================

def calcular_metricas(df, preco_compra=None):
    """
    Calcula:
    - Lucro por Ação (R$): diferença entre o preço atual e o preço de compra
    - Valorização (%): variação percentual em relação ao preço de compra
    """
    if preco_compra is None:
        preco_compra = df['Close'].iloc[0]
        print(f"\nPreço de compra não informado. Usando o primeiro preço do período: R$ {preco_compra:.2f}")
    else:
        print(f"\nPreço de compra informado: R$ {preco_compra:.2f}")

    df['Lucro por Ação (R$)'] = df['Close'] - preco_compra
    df['Valorização (%)'] = ((df['Close'] - preco_compra) / preco_compra) * 100
    return df, preco_compra

# ============================================================
# PLOTAGEM
# ============================================================

def plotar_graficos(df, ticker, preco_compra):
    fig, axes = plt.subplots(2, 1, figsize=(12, 8), sharex=True)

    # Lucro por Ação (R$)
    axes[0].plot(df.index, df['Lucro por Ação (R$)'], color='#2E86AB', linewidth=1.5)
    axes[0].axhline(0, color='gray', linestyle='--', linewidth=0.8)
    axes[0].fill_between(df.index, df['Lucro por Ação (R$)'], 0,
                         where=(df['Lucro por Ação (R$)'] >= 0), color='green', alpha=0.2)
    axes[0].fill_between(df.index, df['Lucro por Ação (R$)'], 0,
                         where=(df['Lucro por Ação (R$)'] < 0), color='red', alpha=0.2)
    axes[0].set_ylabel('Lucro por Ação (R$)')
    axes[0].set_title(f'{ticker} — Lucro por Ação (Preço de Compra: R$ {preco_compra:.2f})',
                      fontsize=13, fontweight='bold')
    axes[0].grid(True, linestyle='--', alpha=0.5)

    # Valorização (%)
    axes[1].plot(df.index, df['Valorização (%)'], color='#A23B72', linewidth=1.5)
    axes[1].axhline(0, color='gray', linestyle='--', linewidth=0.8)
    axes[1].fill_between(df.index, df['Valorização (%)'], 0,
                         where=(df['Valorização (%)'] >= 0), color='green', alpha=0.2)
    axes[1].fill_between(df.index, df['Valorização (%)'], 0,
                         where=(df['Valorização (%)'] < 0), color='red', alpha=0.2)
    axes[1].set_ylabel('Valorização (%)')
    axes[1].set_xlabel('Data')
    axes[1].set_title('Valorização Percentual em relação ao Preço Unitário',
                      fontsize=13, fontweight='bold')
    axes[1].grid(True, linestyle='--', alpha=0.5)

    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

# ============================================================
# INTERAÇÃO COM O USUÁRIO
# ============================================================

def perguntar_ticker():
    """Pergunta o ticker ao usuário, com validação básica."""
    ticker = input("Digite o ticker da ação (ex: PETR4, VALE3, AAPL): ").strip().upper()
    if not ticker:
        print("❌ Ticker não informado.")
        sys.exit(1)
    # Adiciona o sufixo .SA caso o usuário digite apenas o código da B3
    if "." not in ticker and len(ticker) <= 6 and ticker.isalnum():
        ticker_sa = f"{ticker}.SA"
        usar_sa = input(f"Adicionar sufixo '.SA' (mercado B3)? [s/N]: ").strip().lower()
        if usar_sa == "s":
            ticker = ticker_sa
    return ticker

def perguntar_periodo():
    """Pergunta o período desejado com menu."""
    print("\nEscolha o período de análise:")
    print("  1 - 1 ano")
    print("  2 - 2 anos")
    print("  3 - 5 anos")
    print("  4 - Máximo disponível")
    op = input("Opção: ").strip()
    mapa = {"1": "1y", "2": "2y", "3": "5y", "4": "max"}
    return mapa.get(op, "2y")

def perguntar_preco_compra():
    """Pergunta o preço de compra (opcional)."""
    entrada = input("\nPreço unitário de compra (R$) — pressione Enter para usar o primeiro preço do período: ").strip()
    if not entrada:
        return None
    try:
        return float(entrada.replace(",", "."))
    except ValueError:
        print("❌ Valor inválido. Usando o primeiro preço do período.")
        return None

# ============================================================
# FLUXO PRINCIPAL
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("📈 GRÁFICO DE LUCRO POR AÇÃO E VALORIZAÇÃO")
    print("=" * 60)

    ticker = perguntar_ticker()
    periodo = perguntar_periodo()
    preco_compra = perguntar_preco_compra()

    print(f"\n🔍 Baixando dados de {ticker} (período: {periodo})...")
    try:
        df = obter_dados(ticker, periodo)
    except Exception as e:
        print(f"❌ Erro ao buscar dados: {e}")
        print("Verifique se o ticker está correto e se há conexão com a internet.")
        sys.exit(1)

    df, preco_compra = calcular_metricas(df, preco_compra)

    print("\nÚltimos 5 registros:")
    print(df[['Close', 'Lucro por Ação (R$)', 'Valorização (%)']].tail().round(2).to_string())

    plotar_graficos(df, ticker, preco_compra)
