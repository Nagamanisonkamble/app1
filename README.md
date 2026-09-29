# app1import os
import requests
from bs4 import BeautifulSoup
import pandas as pd
from datetime import datetime

class GitHubProjectScraper:
    def __init__(self, target_url: str):
        self.url = target_url
        self.headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        }
        self.data_list = []

    def fetch_page(self):
        """Fetches the webpage content securely."""
        try:
            print(f"[INFO] Fetching content from {self.url}...")
            response = requests.get(self.url, headers=self.headers, timeout=10)
            response.raise_for_status()
            return response.text
        except requests.exceptions.RequestException as e:
            print(f"[ERROR] Failed to retrieve data: {e}")
            return None

    def parse_html(self, html_content):
        """Parses the HTML. (Configured here for a sample news blog layout)"""
        if not html_content:
            return
        
        soup = BeautifulSoup(html_content, 'html.parser')
        
        # Example target: Extracts articles inside common semantic tags
        articles = soup.find_all(['article', 'div'], class_=['post', 'story', 'article-body'])
        
        # Fallback to general headings if no specialized containers are found
        if not articles:
            articles = soup.find_all(['h1', 'h2', 'h3'])

        print(f"[INFO] Found {len(articles)} potential elements to parse.")

        for index, item in enumerate(articles):
            # Extract text safely
            title = item.get_text(strip=True)
            
            # Extract links if available
            link_tag = item.find('a') if hasattr(item, 'find') else None
            link = link_tag['href'] if link_tag and link_tag.has_attr('href') else self.url
            
            if title and len(title) > 5:  # Filter out trivial layout snippets
                self.data_list.append({
                    "ID": index + 1,
                    "Title": title,
                    "Source_URL": link,
                    "Scraped_At": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                })

    def process_and_save(self, filename="scraped_data.csv"):
        """Cleans extracted data with Pandas and saves it to a CSV."""
        if not self.data_list:
            print("[WARN] No data compiled. Saving aborted.")
            return

        # Convert to DataFrame
        df = pd.DataFrame(self.data_list)
        
        # Simple Data Cleaning: Remove duplicate titles
        df.drop_duplicates(subset=['Title'], inplace=True)
        
        # Export
        output_path = os.path.join(os.getcwd(), filename)
        df.to_csv(output_path, index=False)
        print(f"[SUCCESS] Cleaned data saved successfully to: {output_path}")
        print(df.head(5)) # Display preview of first 5 rows

if __name__ == "__main__":
    # Feel free to change this URL to any blog or news portal you want to test
    TARGET = "https://ycombinator.com" 
    
    scraper = GitHubProjectScraper(TARGET)
    html = scraper.fetch_page()
    scraper.parse_html(html)
    scraper.process_and_save("github_scraping_project.csv")
