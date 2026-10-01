import cv2
import numpy as np
import streamlit as st
import tempfile
from ultralytics import YOLO

# Configuração da página
st.set_page_config(page_title="Detector YOLO", layout="wide")

@st.cache_resource
def load_model():
    return YOLO("yolov8n.pt")

model = load_model()

st.title("Detecção de Objetos em Tempo Real")
st.sidebar.header("Configurações")

confidence = st.sidebar.slider("Limiar de Confiança", 0.1, 1.0, 0.5, 0.05)
option = st.sidebar.selectbox("Fonte de Entrada", ["Imagem", "Vídeo"])

# --- PROCESSAMENTO DE IMAGEM ---
if option == "Imagem":
    uploaded_file = st.file_uploader("Envie uma imagem", type=["jpg", "jpeg", "png"])
    
    if uploaded_file is not None:
        file_bytes = np.asarray(bytearray(uploaded_file.read()), dtype=np.uint8)
        image = cv2.imdecode(file_bytes, cv1.IMREAD_COLOR if hasattr(cv2, 'IMREAD_COLOR') else 1)
        
        # Realiza a predição
        results = model.predict(source=image, conf=confidence)
        
        # Desenha as detecções e converte BGR para RGB
        res_plotted = results[0].plot()
        res_rgb = cv2.cvtColor(res_plotted, cv2.COLOR_BGR2RGB)
        
        st.image(res_rgb, caption="Resultado da Detecção", use_container_width=True)

# --- PROCESSAMENTO DE VÍDEO ---
elif option == "Vídeo":
    uploaded_video = st.file_uploader("Envie um vídeo", type=["mp4", "avi", "mov"])
    
    if uploaded_video is not None:
        # Salva o arquivo temporariamente pois o OpenCV precisa de um caminho no disco
        tfile = tempfile.NamedTemporaryFile(delete=False)
        tfile.write(uploaded_video.read())
        
        cap = cv2.VideoCapture(tfile.name)
        st_frame = st.empty()
        
        stop_button = st.button("Interromper")
        
        while cap.isOpened() and not stop_button:
            ret, frame = cap.read()
            if not ret:
                break
            
            # Processa cada frame
            results = model.predict(source=frame, conf=confidence)
            res_plotted = results[0].plot()
            res_rgb = cv2.cvtColor(res_plotted, cv2.COLOR_BGR2RGB)
            
            # Atualiza o frame na tela
            st_frame.image(res_rgb, caption="Processando Vídeo...", use_container_width=True)
            
        cap.release()