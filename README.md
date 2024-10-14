import time
import pandas as pd
from bs4 import BeautifulSoup
from textblob import TextBlob
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

def setup_driver():
    options = Options()
    options.headless = True
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)
    return driver

def scrape_reviews(product_url):
    driver = setup_driver()
    driver.get(product_url)
    time.sleep(3)
    soup = BeautifulSoup(driver.page_source, 'html.parser')
    driver.quit()

    reviews = []
    for review in soup.find_all('div', class_='review'):
        rating = review.find('div', class_='rating').get_text(strip=True)
        review_text = review.find('p', class_='review-text').get_text(strip=True)
        reviews.append({'Rating': rating, 'Review Text': review_text})
    return reviews

def analyze_sentiment(review_text):
    analysis = TextBlob(review_text)
    return analysis.sentiment.polarity

def main():
    product_urls = [
        'https://www.flipkart.com/samsung-galaxy-m53-5g-graphite-128-gb/p/itm3cc3b12a0572d',
        'https://www.flipkart.com/realme-9-5g-supersonic-blue-128-gb/p/itm5a81808af436f',
        'https://www.flipkart.com/vivo-t1-5g-starlight-black-128-gb/p/itm621cda38f65f3',
        'https://www.flipkart.com/oneplus-nord-2t-5g-grey-sierra-128-gb/p/itm7de462ae0c49d',
        'https://www.flipkart.com/haier-hwm65-707nzp-6-5-kg-fully-automatic-top-load-washing-machine/p/itm52b4e0c02e31b',
    ]
    
    all_reviews = []
    for url in product_urls:
        reviews = scrape_reviews(url)
        all_reviews.extend(reviews)
        time.sleep(2)

    df = pd.DataFrame(all_reviews)
    df['Sentiment Score'] = df['Review Text'].apply(analyze_sentiment)
    df['Sentiment'] = df['Sentiment Score'].apply(lambda x: 'Positive' if x > 0 else ('Negative' if x < 0 else 'Neutral'))
    
    df.to_csv('scraped_reviews.csv', index=False)
    print(df)

main()
