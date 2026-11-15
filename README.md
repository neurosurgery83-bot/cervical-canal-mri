Cervical Canal MRI Measurement Assistant
Run:
pip install -r requirements.txt
streamlit run app.py
Open the Streamlit URL on your computer. To use your phone camera, deploy the app on an HTTPS host (for example Streamlit Community Cloud) and open the secure URL on your phone. A local HTTP LAN URL may not permit camera access on mobile browsers.
Use de-identified images only. No server-side database or persistent image storage is implemented. Uploaded images are still transmitted to the Streamlit server hosting the app; use an approved secure environment for clinical data.
Tap two endpoints per measurement. Calibrate with a reliable image scale to report millimeters. A photograph without calibration is not suitable for absolute measurements. This is a manual measurement aid, not automated image segmentation or diagnosis.
