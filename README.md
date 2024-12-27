# 필요한 라이브러리 설치
!pip install requests beautifulsoup4 pandas googletrans==4.0.0-rc1

import requests
from bs4 import BeautifulSoup
import pandas as pd
from googletrans import Translator
import time
import random

# Google Translator 객체 생성
translator = Translator()

# 기본 URL과 UID 설정
base_url = "https://www.gohackers.com/?c=village/village_info/eng_edu/toefl_background&type=view&uid={uid}"
uids = range(88, 91)  # UID 값은 원하는 범위로 설정 가능

# 데이터를 저장할 리스트 초기화
data = []

def fetch_data(uid):
    """
    주어진 UID로부터 데이터를 크롤링하는 함수
    """
    url = base_url.format(uid=uid)
    response = requests.get(url)
    response.raise_for_status()  # HTTP 에러 처리
    soup = BeautifulSoup(response.text, "html.parser")
    
    # Title 추출
    title_element = soup.find("h3", {"class": "tit"})
    title = title_element.get_text(strip=True) if title_element else "No Title"

    # Content 추출
    content_element = soup.find("p", {"class": "txt"})
    content = content_element.get_text(strip=True) if content_element else "No Content"

    # Word 리스트 추출
    word_elements = soup.find("ul", {"class": "lst_relation"})
    words = [li.get_text(strip=True) for li in word_elements.find_all("li")] if word_elements else []

    return title, content, words

def translate_text(text):
    """
    텍스트를 영어로 번역하는 함수
    """
    try:
        return translator.translate(text, src='ko', dest='en').text
    except Exception as e:
        print(f"번역 실패: {e}")
        return text  # 실패 시 원본 텍스트 반환

def random_sleep():
    """
    1초에서 3초 사이의 랜덤 대기 시간을 적용
    """
    sleep_time = random.uniform(1, 3)
    print(f"대기 시간: {sleep_time:.2f}초")
    time.sleep(sleep_time)

# UID를 변경하며 크롤링
for uid in uids:
    try:
        # 데이터 가져오기
        title, content, words = fetch_data(uid)
        translated_content = translate_text(content)

        # 결과 저장
        data.append({
            "UID": uid,
            "Title": title,
            "Content": content,
            "Translated Content": translated_content,  # 번역된 내용 추가
            "Words": ", ".join(words)  # 단어를 콤마로 구분하여 저장
        })
        print(f"UID {uid} 처리 성공: {title}")

        # 랜덤 대기
        random_sleep()

    except requests.exceptions.RequestException as e:
        print(f"네트워크 에러 - UID {uid}: {e}")
    except Exception as e:
        print(f"데이터 처리 에러 - UID {uid}: {e}")

# 데이터프레임으로 변환 후 저장
df = pd.DataFrame(data)
print(df)
df.to_csv("toefl_background_data_translated.csv", index=False, encoding="utf-8-sig")
print("크롤링 및 번역 데이터 저장 완료: toefl_background_data_translated.csv")
