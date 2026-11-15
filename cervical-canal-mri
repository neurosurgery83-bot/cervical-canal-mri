import io
import streamlit as st
from PIL import Image, ImageOps, ImageDraw
from streamlit_image_coordinates import streamlit_image_coordinates

st.set_page_config(page_title="Cervical Canal MRI Measurement", page_icon="🧠", layout="wide")
st.title("Cervical Canal MRI Measurement Assistant")
st.caption("Educational measurement aid • Not an autonomous diagnostic or medical device • Use de-identified images only")

if "points" not in st.session_state: st.session_state.points = []
if "measurements" not in st.session_state: st.session_state.measurements = {}
if "image_id" not in st.session_state: st.session_state.image_id = None

with st.expander("How to use / anatomical guidance", expanded=True):
    st.markdown("""
1. Upload a **PNG/JPG/JPEG** or use your phone camera. Prefer a straight **midsagittal MRI** screenshot with the vertebral body and posterior bony canal clearly visible.
2. Select a measurement and tap its **two endpoints**. For AP canal diameter, measure from the posterior vertebral-body cortex to the anterior laminar/spinolaminar boundary **at the vertebral-body level**, not at a disc.
3. For cord AP diameter, measure the anterior-to-posterior cord dimension at the **same noncompressed level**. SCOR = cord AP / canal AP × 100.
4. For millimeters, calibrate using a **known distance in the same image** (e.g., a trustworthy on-image scale bar). A photo of a screen or screenshot without calibration **cannot provide valid millimeter measurements**.
5. Check axial images, CSF effacement, degenerative disease, and cord signal independently. Confirm bony landmarks on source MRI/CT if uncertain.

**Important:** A phone photograph may have perspective distortion, which makes measurements unreliable. Avoid identifiable patient information; images are processed in the active app session, not saved by this app.
""")

source=st.radio("Image source", ["Upload image", "Phone camera"], horizontal=True)
file = st.file_uploader("Select MRI image", type=["png","jpg","jpeg"]) if source=="Upload image" else st.camera_input("Photograph MRI display")
if not file:
    st.info("Add an MRI image to begin.")
    st.stop()
raw=file.getvalue()
import hashlib
image_id=hashlib.sha256(raw).hexdigest()
if st.session_state.image_id != image_id:
    st.session_state.image_id=image_id
    st.session_state.points=[]
    st.session_state.measurements={}
image=ImageOps.exif_transpose(Image.open(io.BytesIO(raw))).convert("RGB")
w,h=image.size
display_w=min(w, 900)
scale=display_w/w
display_h=round(h*scale)

level=st.selectbox("Vertebral body level", ["C3","C4","C5","C6","C7"])
kind=st.selectbox("Select measurement", ["Calibration reference", "Canal AP", "Cord AP"])
if st.button("Clear current taps"):
    st.session_state.points=[]
    st.rerun()

# Draw annotations on a copy, with coordinate system matched to displayed image.
canvas=image.resize((display_w,display_h))
draw=ImageDraw.Draw(canvas)
for name, (a,b) in st.session_state.measurements.items():
    if name.startswith(level+":") or name=="Calibration reference":
        color="#24D4B0" if "Canal" in name else "#FFCB57" if "Cord" in name else "#F2788C"
        aa=(round(a[0]*scale),round(a[1]*scale))
        bb=(round(b[0]*scale),round(b[1]*scale))
        draw.line([aa,bb], fill=color, width=3)
        for x,y in [aa,bb]: draw.ellipse((x-5,y-5,x+5,y+5),fill=color)
for p in st.session_state.points:
    x,y=round(p[0]*scale),round(p[1]*scale)
    draw.ellipse((x-5,y-5,x+5,y+5),fill="red")
st.write("Tap two endpoints on the image (one at a time).")
clicked=streamlit_image_coordinates(canvas,key=f"img_{image_id}_{level}_{kind}_{len(st.session_state.points)}_{len(st.session_state.measurements)}")
if clicked:
    point=(clicked["x"]/scale,clicked["y"]/scale)
    st.session_state.points.append(point)
    if len(st.session_state.points)==2:
        key="Calibration reference" if kind=="Calibration reference" else f"{level}:{kind}"
        st.session_state.measurements[key]=tuple(st.session_state.points)
        st.session_state.points=[]
    st.rerun()

import math
def distance(pair):
    return math.dist(pair[0],pair[1])
cal=st.session_state.measurements.get("Calibration reference")
reference_mm=st.number_input("Known calibration distance (mm), only if reference was measured",min_value=0.0,value=0.0,step=1.0)
mm_per_pixel=reference_mm/distance(cal) if cal and reference_mm>0 and distance(cal)>0 else None
if not mm_per_pixel: st.warning("No valid mm calibration. SCOR can still be computed from pixel ratios if canal and cord are measured on the same undistorted image.")

thresholds={"C3":12.9,"C4":11.8,"C5":11.9,"C6":12.3,"C7":13.3}
rows=[]
for lvl in ["C3","C4","C5","C6","C7"]:
    ca=st.session_state.measurements.get(f"{lvl}:Canal AP")
    co=st.session_state.measurements.get(f"{lvl}:Cord AP")
    ca_px=distance(ca) if ca else None
    co_px=distance(co) if co else None
    ca_mm=ca_px*mm_per_pixel if ca_px and mm_per_pixel else None
    scor=100*co_px/ca_px if ca_px and co_px else None
    rows.append({"Level":lvl,"Canal AP (mm)":round(ca_mm,2) if ca_mm else None,
                 "Cord AP (mm)":round(co_px*mm_per_pixel,2) if co_px and mm_per_pixel else None,
                 "SCOR (%)":round(scor,1) if scor is not None else None,
                 "Research cutoff (mm)":thresholds[lvl],
                 "Below cutoff?":("Yes" if ca_mm<thresholds[lvl] else "No") if ca_mm is not None else "Unknown"})
st.subheader("Measurements")
st.dataframe(rows,use_container_width=True,hide_index=True)
valid=[r for r in rows if r["Canal AP (mm)"] is not None]
below=sum(r["Below cutoff?"]=="Yes" for r in valid)
if len(valid)==5:
    st.info(f"{below}/5 levels fall below the published level-specific research cutoffs. Three or more is a research-derived multilevel definition, NOT a standalone clinical diagnosis.")
else:
    st.info("Measure and calibrate all C3–C7 levels to assess the five-level research definition.")
current=next(r for r in rows if r["Level"]==level)
if current["SCOR (%)"] is not None:
    st.metric(f"{level} sagittal SCOR",f'{current["SCOR (%)"]:.1f}%')
    if current["SCOR (%)"]>=70:
        st.warning("SCOR ≥70%: increased cord occupation; interpret only at a noncompressed vertebral level and in clinical context.")
st.caption("Published AP cutoffs: C3 <12.9, C4 <11.8, C5 <11.9, C6 <12.3, C7 <13.3 mm. Cutoffs are research-derived and not universally validated.")
st.caption("SCOR ≥70% is a commonly cited research criterion; measurements do not establish myelopathy or treatment indications.")
st.markdown("References: [Congenital Cervical Stenosis review](https://pmc.ncbi.nlm.nih.gov/articles/PMC10427602/) · [MRI research](https://pubmed.ncbi.nlm.nih.gov/39481447/)")
