Crawling asteroid(1 particular sol) lightcurve & 3d obj code:
```py
import requests
from bs4 import BeautifulSoup
import os

#DAMIT database의 소행성 목록 페이지 html 형식이 table이래요. 말그대로 표를 나타내는 html 형식이라는데, 그 안에서 각 행은 <tr>로 표시되고, 각 행의 헤더는 <th>로, 각 행의 cell(열)은 <td>로 표시된대요
#그리고 html 파일에서 어떤 ... 링크를 지정할 때는 <a href="링크 url"> 이런 식으로 쓴다고 합니다.

# 저장할 디렉토리 (D: 외장 SSD)
output_dir = "D:/DAMIT_dataset"
os.makedirs(output_dir, exist_ok=True)
input_training_domain_dir = os.path.join(output_dir, "input_training_domain") # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 input training domain
output_data_1_dir = os.path.join(output_dir, "output_data_1") # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 output data - shape.txt 파일 받음
output_data_2_dir = os.path.join(output_dir, "output_data_2") # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 output data - shape.obj 파일 받음

# 하위 디렉토리 생성
os.makedirs(input_training_domain_dir, exist_ok=True) # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 input training domain
os.makedirs(output_data_1_dir, exist_ok=True) # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 output data - shape.txt 파일 받음
os.makedirs(output_data_2_dir, exist_ok=True) # 외장 SSD의 DAMIT_dataset 폴더의 하위 폴더 output data - shape.obj 파일 받음

# 테스트할 페이지 수 (1~2)
# DAMIT 데이터베이스 들어가보면, 소행성이 목록화되어있고 각각의 모델이 그 아래 행에 정리되어 있음. 그렇게 된 페이지가 총 717페이지인데, 여기서는 테스트용으로 2페이지까지만 하는거.
# 여기에 입력하는 페이지수까지 크롤링(?)함
test_pages = 2

# 첫 번째 페이지 URL
# DAMIT database 첫번째 페이지는 아래와 같은 링크 주소인데, 2페이지부터는 이 뒤에 /page:(페이지번호)가 붙는 반면 첫페이지엔 안 붙음. 그래서 첫번째만 따로 해줌.
first_page_url = "https://astro.troja.mff.cuni.cz/projects/damit/asteroids/browse/sort:Asteroid.number/direction:asc"

# 모든 페이지를 순회
for page_number in range(1, test_pages + 1):
    if page_number == 1:
        url = first_page_url
    else:
        url = f"https://astro.troja.mff.cuni.cz/projects/damit/asteroids/browse/sort:Asteroid.number/direction:asc/page:{page_number}"

    print(f"Requesting URL: {url}")  # 요청 URL 출력
    response = requests.get(url)

    # 요청 상태 확인
    if response.status_code != 200:
        print(f"Failed to retrieve page {page_number}. Status code: {response.status_code}")
        continue

    soup = BeautifulSoup(response.text, "html.parser")
    print("Page retrieved successfully!")  # 페이지 성공적으로 가져옴

    # 소행성 이름과 모델 링크 저장
    asteroid_rows = soup.find_all('tr', class_='damit-asteroid-row')

    for asteroid_row in asteroid_rows:
        # 소행성 이름 가져오기
        asteroid_name = asteroid_row.find('th').get_text(strip=True)  # <th>에서 이름 가져오기. asteroid_row 행의 헤더에 그 소행성의 이름이 있음. 파일 저장할 때 그 이름을 쓸거라 여기서 확인함.
        print(f"Found asteroid: {asteroid_name}")  # 페이지에서 찾은 소행성 이름 출력

        # 방금 찾은 소행성에 딸려있는 모델 개수 셀거임.
        model_row_count = 0
        current_row = asteroid_row

        while True:
            # 다음 행으로 넘어가기
            next_row = current_row.find_next_sibling('tr')

            # 모델 행 찾기
            if next_row and 'damit-model-row' in next_row.get('class', []): #모델이 있는 행인 경우
                model_row_count += 1 #하나의 소행성에 대해 몇개의 모델이 있는지 셈.
                current_row = next_row
            else: #새로운 소행성이 있는 행인 경우
                break

        # 특이해가 존재하는 경우 (모델이 1개인 경우)
        if model_row_count == 1:
            print(f"Unique model found for asteroid: {asteroid_name}") #특이해가 존재하는 소행성을 찾았다고 출력

            # lc.txt 링크 찾기: 첫 번째 <td>의 두 번째 <a> 태그에 들어있음.
            lc_links = asteroid_row.find_all('td')[0].find_all('a')  # 첫 번째 <td>의 모든 <a> 태그
            if len(lc_links) > 1:
                lc_url = lc_links[1]['href']  # 두 번째 <a> 태그의 href: 여기에 lc.txt 나오는 링크 있음.
                if not lc_url.startswith('http'):
                    lc_url = "https://astro.troja.mff.cuni.cz" + lc_url #링크가 상대경로로 주어지므로, base url과 결합해야 온전한 주소.

                print(f"Downloading lc.txt from: {lc_url}")
                lc_response = requests.get(lc_url)
                lc_filename = os.path.join(input_training_domain_dir, f"{asteroid_name}_lc.txt") # 첨에 만든 input 데이터 폴더에 저장합니다
                with open(lc_filename, 'w') as f:
                    f.write(lc_response.text)
                print(f"Saved: {lc_filename}")
            else:
                print("lc.txt link not found!")

            # shape.txt 및 shape.obj 링크 찾기: 얘네는 model row의 5번째 td에 있음.
            shape_link = current_row.find_all('td')[4].find('a', href=lambda href: href and "shape.txt" in href) #5번째 td에 있는 a 태그 중에, href에 shape.txt가 포함된 것
            obj_link = current_row.find_all('td')[4].find('a', href=lambda href: href and "shape.obj" in href) #5번째 td에 있는 a 태그 중에, href에 shape.obj가 포함된 것

            # shape.txt 다운로드
            if shape_link:
                shape_url = shape_link['href']
                if not shape_url.startswith('http'):
                    shape_url = "https://astro.troja.mff.cuni.cz" + shape_url

                print(f"Downloading shape.txt from: {shape_url}")
                shape_response = requests.get(shape_url)
                shape_filename = os.path.join(output_data_1_dir, f"{asteroid_name}_shape.txt") # 첨에 만든 output1 데이터 폴더에 저장합니다
                with open(shape_filename, 'w') as f:
                    f.write(shape_response.text)
                print(f"Saved: {shape_filename}")
            else:
                print("shape.txt link not found!")

            # shape.obj 다운로드
            if obj_link:
                obj_url = obj_link['href']
                if not obj_url.startswith('http'):
                    obj_url = "https://astro.troja.mff.cuni.cz" + obj_url

                print(f"Downloading shape.obj from: {obj_url}")
                obj_response = requests.get(obj_url)
                obj_filename = os.path.join(output_data_2_dir, f"{asteroid_name}_shape.obj") # 첨에 만든 output2 데이터 폴더에 저장합니다
                with open(obj_filename, 'w') as f:
                    f.write(obj_response.text)
                print(f"Saved: {obj_filename}")
            else:
                print("shape.obj link not found!")

print("모든 페이지에서 파일 다운로드 완료!") # 모든 페이지 크롤링(?) 완료하면 끄읕

```
