from fastapi import FastAPI, File, UploadFile
from fastapi.responses import JSONResponse
from some_module import enhance_image, create_prompt, generate_content_stream, logger

app = FastAPI()

# Map review types to template file paths
TEMPLATE_MAPPING = {
    "General Architecture Review": "prompts/ada/general_architecture",
    "Security Architecture Review": "prompts/ada/security_architecture",
    "Infrastructure Architecture Review": "prompts/ada/infrastructure_architecture",
    "Application Architecture Review": "prompts/ada/application_architecture",
}

@app.post("/analyze_diagram")
async def analyze_diagram(selected_template: str, uploaded_file: UploadFile = File(...)):
    # Step 1: Map the review type to the template path
    template_path = TEMPLATE_MAPPING.get(selected_template)
    if not template_path:
        return JSONResponse(content={"error": "Invalid review type selected."}, status_code=400)

    # Step 2: Read the template content
    template_content = read_prompt_template(template_path)
    if not template_content:
        return JSONResponse(content={"error": "Template not found."}, status_code=404)

    # Step 3: Process the uploaded file
    image_data = await uploaded_file.read()
    try:
        enhanced_image = enhance_image(image_data)  # Enhance the image
    except Exception as e:
        return JSONResponse(content={"error": f"Image enhancement failed: {str(e)}"}, status_code=500)

    # Step 4: Generate the prompt and model response
    try:
        prompt = create_prompt(template_content)
        response, hyperparameters = generate_content_stream(prompt, enhanced_image)  # Generate response
    except Exception as e:
        return JSONResponse(content={"error": f"Model generation failed: {str(e)}"}, status_code=500)

    # Step 5: Log the results
    logger.log_lim_run(
        schema_name="experiment_name",
        prompt=prompt,
        model_response=response
    )

    # Step 6: Return the response
    return {"response": response, "hyperparameters": hyperparameters}
