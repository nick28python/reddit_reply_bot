from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# --- YOUR DETAILS ---
USERNAME = "your_reddit_username"
PASSWORD = "your_reddit_password"
POST_URL = "https://www.reddit.com/r/test/comments/xxxxxx/"  # Replace with actual post URL
REPLY_TEXT = "This is an automated reply from my Python bot 🤖"

# --- CHROME OPTIONS ---
options = Options()
# Comment this during testing (headless = no browser shown)
# options.add_argument("--headless")
options.add_argument("--start-maximized")
options.add_experimental_option("detach", True)  # Keeps browser open

# --- START DRIVER ---
driver = webdriver.Chrome(options=options)
wait = WebDriverWait(driver, 20)

# --- STEP 1: GO TO LOGIN PAGE ---
driver.get("https://www.reddit.com/login")
time.sleep(3)

# --- STEP 2: ACCEPT COOKIE POPUP IF ANY ---
try:
    accept_btn = WebDriverWait(driver, 5).until(
        EC.element_to_be_clickable((By.XPATH, "//button[contains(text(), 'Accept all')]"))
    )
    accept_btn.click()
    print("✅ Accepted cookies popup")
except:
    print("🔸 No cookie popup found")

# --- STEP 3: LOGIN FORM FILL ---
try:
    username_box = wait.until(EC.presence_of_element_located((By.ID, "loginUsername")))
    username_box.send_keys(USERNAME)
    
    password_box = driver.find_element(By.ID, "loginPassword")
    password_box.send_keys(PASSWORD)

    login_button = driver.find_element(By.XPATH, "//button[contains(text(), 'Log In')]")
    login_button.click()
    print("🔐 Logging in...")

    # Wait for homepage redirect or profile icon to appear
    time.sleep(10)

except Exception as e:
    print("❌ Login failed:", e)
    driver.quit()
    exit()

# --- STEP 4: GO TO TARGET POST ---
driver.get(POST_URL)
time.sleep(5)

# --- STEP 5: SCROLL TO BOTTOM TO LOAD REPLY BOX ---
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
time.sleep(5)

# --- STEP 6: REPLY TO POST ---
try:
    reply_box = wait.until(EC.presence_of_element_located(
        (By.CSS_SELECTOR, "div[data-test-id='comment-rich-text-editor'] div[role='textbox']")
    ))
    reply_box.click()
    reply_box.send_keys(REPLY_TEXT)
    time.sleep(2)

    reply_button = driver.find_element(By.XPATH, "//button[contains(text(), 'Reply')]")
    reply_button.click()
    print("✅ Replied successfully!")

except Exception as e:
    print("❌ Failed to reply:", e)

# Optional: Keep browser open to confirm visually
time.sleep(10)
driver.quit()
