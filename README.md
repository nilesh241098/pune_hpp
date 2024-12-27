# pune_hpp
import streamlit as st
import requests

# Define the FastAPI backend URL
API_URL = "http://127.0.0.1:8000/analyze_diagram"

# Layout for the Streamlit app
st.title("Architecture Diagram Review")

# Step 1: File Upload
uploaded_file = st.file_uploader("Upload a diagram", type=["png", "jpg", "jpeg"])

# Step 2: Template Selection
selected_template = st.selectbox(
    "Select Review Type",
    [
        "General Architecture Review",
        "Security Architecture Review",
        "Infrastructure Architecture Review",
        "Application Architecture Review",
    ]
)

# Step 3: Display the uploaded image
if uploaded_file is not None:
    st.image(uploaded_file, caption="Uploaded Diagram", use_column_width=True)

# Step 4: Analyze button
if st.button("Analyze Diagram"):
    if uploaded_file is not None:
        # Prepare the file and data to send to the backend
        files = {"uploaded_file": (uploaded_file.name, uploaded_file, uploaded_file.type)}
        data = {"selected_template": selected_template}

        # Send the request to the FastAPI backend
        with st.spinner("Analyzing the diagram..."):
            response = requests.post(API_URL, data=data, files=files)

        # Handle the response
        if response.status_code == 200:
            result = response.json()
            st.markdown("---")
            st.markdown(result["response"])  # Display the response
            st.download_button(
                label="Download Review Comments",
                data=result["response"],
                file_name=f"ADA_{selected_template}.md",
                mime="text/markdown",
            )
        else:
            error_message = response.json().get("error", "An error occurred.")
            st.error(f"Failed to analyze the diagram: {error_message}")
    else:
        st.warning("Please upload a diagram first.")
