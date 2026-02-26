import streamlit as st
from openai import AzureOpenAI

# --- 1. Security & Password Check ---
def check_password():
    def password_entered():
        if st.session_state["password"] == st.secrets["APP_PASSWORD"]:
            st.session_state["password_correct"] = True
            del st.session_state["password"]
        else:
            st.session_state["password_correct"] = False

    if "password_correct" not in st.session_state:
        st.text_input("Enter Password to Unlock AI", type="password", on_change=password_entered, key="password")
        return False
    elif not st.session_state["password_correct"]:
        st.text_input("Enter Password to Unlock AI", type="password", on_change=password_entered, key="password")
        st.error("😕 Password incorrect")
        return False
    return True

if not check_password():
    st.stop()

# --- 2. Client Setup ---
try:
    client = AzureOpenAI(
        api_key=st.secrets["AZURE_OPENAI_KEY"],
        api_version="2023-05-15", # Updated to a standard stable version
        azure_endpoint=st.secrets["AZURE_OPENAI_ENDPOINT"]
    )
    deployment_name = st.secrets["DEPLOYMENT_NAME"]
except Exception as e:
    st.error(f"⚠️ API Configuration error: {e}")
    st.stop()

# --- 3. The Main App ---
st.title("💼 Freelance AI Workspace")

with st.sidebar:
    st.header("Settings")
    system_role = st.text_area("AI Role:", "You are a professional business consultant.")
    max_tokens = st.slider("Response Length", 50, 2000, 500)
    temperature = st.slider("Creativity (Temperature)", 0.0, 1.0, 0.7)
    if st.button("Clear Chat"):
        st.session_state.messages = []
        st.rerun()

if "messages" not in st.session_state:
    st.session_state.messages = []

# Display chat history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Logic for New Message
if prompt := st.chat_input("Ask anything..."):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    try:
        with st.chat_message("assistant"):
            # Include the system role for personality
            full_context = [{"role": "system", "content": system_role}] + st.session_state.messages
            
            completion = client.chat.completions.create(
                model=deployment_name, 
                messages=full_context,
                max_tokens=max_tokens,
                temperature=temperature
            )
            
            response = completion.choices[0].message.content
            st.markdown(response)
            st.session_state.messages.append({"role": "assistant", "content": response})
    except Exception as e:
        st.error(f"Error generating response: {e}")
