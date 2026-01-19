# karate-appimport streamlit as st
import graphviz
import math
import random

# --- 基本設定 ---
st.title("🥋 空手トーナメント保存機能付き")

if 'players' not in st.session_state:
    st.session_state.players = []

# --- 選手入力セクション ---
players_input = st.text_area("選手名を改行区切りで入力してください", height=150)
if st.button("トーナメント表を生成"):
    st.session_state.players = [p.strip() for p in players_input.split("\n") if p.strip()]

if st.session_state.players:
    players = st.session_state.players.copy()
    num_players = len(players)
    
    # トーナメントの枠組み計算
    next_pow2 = 2**math.ceil(math.log2(num_players))
    temp_players = players + ["(不戦勝)"] * (next_pow2 - num_players)
    
    # --- Graphvizによる図の生成 ---
    dot = graphviz.Digraph(format='png') # ここを 'pdf' に変えればPDF出力可能
    dot.attr(rankdir='LR', fontname="MS Gothic") # 日本語フォント指定
    dot.attr('node', fontname="MS Gothic", shape="box", style="rounded")

    # 1回戦のノード作成
    matches = []
    for i in range(0, len(temp_players), 2):
        m_id = f"R1_M{i//2}"
        label = f"{temp_players[i]}\nvs\n{temp_players[i+1]}"
        dot.node(m_id, label)
        matches.append(m_id)

    # 2回戦以降の枠（空の枠）を自動生成
    current_round = matches
    round_num = 2
    while len(current_round) > 1:
        next_round = []
        for i in range(0, len(current_round), 2):
            m_id = f"R{round_num}_M{i//2}"
            dot.node(m_id, f"第{round_num}回戦\n(勝者)")
            dot.edge(current_round[i], m_id)
            if i+1 < len(current_round):
                dot.edge(current_round[i+1], m_id)
            next_round.append(m_id)
        current_round = next_round
        round_num += 1

    # --- 画面表示 ---
    st.header("生成されたトーナメント表")
    st.graphviz_chart(dot)

    # --- ダウンロード機能 ---
    # renderメソッドで画像ファイルとして保存
    output_path = "tournament_chart"
    dot.render(output_path, cleanup=True) # tournament_chart.png が生成される

    with open(f"{output_path}.png", "rb") as file:
        btn = st.download_button(
            label="🖼️ 画像(PNG)としてダウンロード",
            data=file,
            file_name="karate_tournament.png",
            mime="image/png"
        )
    
    st.caption("※ PDF版が必要な場合は、コード内の format='png' を 'pdf' に書き換えてください。")

else:
    st.info("選手名を入力して生成ボタンを押してください。")
