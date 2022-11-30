# Jenkins

## Table of contents
- [概述](#概述)
- [用 Docker 安裝 Jenkins](#用-Docker-安裝-Jenkins)
- [建立 Jenkins 作業](#建立-Jenkins-作業)
- [連結 Github webhook 完成 CI 流程](#連結-Github-webhook-完成-CI-流程)
  * [將 Jenkins 連結至外部網址](#將-Jenkins-連結至外部網址)
  * [在 Github 中建立 Webhook](#在-Github-中建立-Webhook)
- [參考資料](#參考資料)

## 概述

<img src="https://i.imgur.com/FIjlpfE.png" alt="Graph" width="75%"/>

## 用 Docker 安裝 Jenkins

1. 下載 Images。

    ![](https://i.imgur.com/UIRK4v5.png)
    ``` =
    docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins:lts-jdk11
    ```
    **指令說明**
    - `docker run` 啟動容器，並把 docker images 放進去跑，這裡叫 `jenkins/jenkins`
    - `-d` 表示放在背景執行
    - `-v` 表示 volumne，創建一個叫 jenkins_home 的新 volumne，把它 Mapping 到 Container 裡面的 `/var/jenkins_home` 這個路徑，目的是讓我們的 Jenkins 的設定，在我們關掉重啟 Container 之後都還能繼續存在。
    - `-p` 表示 Port Mapping，開了兩個 8080 與 50000

2. 完成後可以執行以下，確認是否成功。

    - `docker images` 查看 images
    - `docker volume ls` 查看 volume
    - `docker ps` 正在運行的 container
    
    ![](https://i.imgur.com/Jehz9Xd.png)


3. 第一次進去時需要一個 Token，可以從 logs 中獲得。
   
   - `docker logs <container id>`，從上步驟得知 container id 為 `84bfeb0fd2a6`，因此執行 `docker logs 84bfeb0fd2a6`
   - 找尋一段 `Please use the following password to proceed to installation` 的字眼，並紀錄下來，如下圖。

    ![](https://i.imgur.com/RWtUKkq.png)

4. 瀏覽器開啟 http://127.0.0.1:8080/ ，並輸入上步驟記下來的密碼。

    <img src="https://i.imgur.com/L0DKBKj.png" alt="Graph" width="55%"/>

5. 點擊`安裝推薦的外掛`就會開始安裝。

    <img src="https://i.imgur.com/hAIOWLf.png" alt="Graph" width="55%"/>


6. 安裝完成後，會跳出建立第一個系統管理員的畫面。

    <img src="https://i.imgur.com/LnDAL8k.png" alt="Graph" width="55%"/>

7. 輸入後就完成安裝了

    <img src="https://i.imgur.com/uucGgd1.png" alt="Graph" width="55%"/>


## 建立 Jenkins 作業
1. 登入 Jenkins。
  
      <img src="https://i.imgur.com/J4Dr3U3.png" alt="Graph" width="55%"/>

2. 點擊`新增作業`。

    ![](https://i.imgur.com/zxDsS66.png)

3. `輸入項目名稱` > `建置 Free-Style 軟體專案` > `確定`。

    <img src="https://i.imgur.com/CBi6BQU.png" alt="Graph" width="65%"/>

4. 至 Github 中複製我們會 clone 下來的網址。

    ![](https://i.imgur.com/Ztj48AJ.png)

5. `原始碼管理` > `Git` > 輸入上步複製的 `Repositoy URL` > 將 Branch 改為 `*/main`（從上圖中的左上角可以看到 branch 是 `main`）> `儲存`。

    ![](https://i.imgur.com/SJlmlG7.png)


6. `建置觸發程序` > 勾選 `GitHub hook trigger for GITScm polling`。

    ![](https://i.imgur.com/UOVSlOA.png)

7. `Build Steps` > 點選 `執行 Shell`，表示收到 Github Events 時要做的事情。

    ![](https://i.imgur.com/2toqbFt.png)

8. 假設這邊輸入 `echo "event from github"`，表示收到 Github Events 時會執行這行，最後點擊`儲存`。

    ![](https://i.imgur.com/s21S91x.png)


## 連結 Github webhook 完成 CI 流程
### 將 Jenkins 連結至外部網址
1. 至 https://ngrok.com/download 下載並解壓縮。

    <img src="https://i.imgur.com/Dfy2XLo.png" alt="Graph" width="65%"/>


2. 開啟 terminal 執行以下。
    - `YourPath/./ngrok http Yourport`，由於位置在下載的資料夾中且 port 為 8080，因此執行 `Downloads/./ngrok http 8080`

3. 就可以得到一個外部網址。

    ![](https://i.imgur.com/b08nzRG.png)

4. 打開上面截圖的網址 https://a9a5-1-163-41-21.ngrok.io ，有出現登入 Jenkins 的畫面表示成功。

    <img src="https://i.imgur.com/dTJEajj.png" alt="Graph" width="65%"/>

### 在 Github 中建立 Webhook
1. 回到 Github 的 Repo 中，`Setting` > `Webhooks` > `Add webhook`。

    ![](https://i.imgur.com/Fw3GXXe.png)

2. 在 `Payload URL` 的地方輸入 ngrok 的網址+`/github-webhook/`， 因此輸入 `https://a9a5-1-163-41-21.ngrok.io/github-webhook/`，接著按 `Add webhook`。

    ![](https://i.imgur.com/2PmPwO7.png)

3. 回到 Code 的頁面，修改檔案來測試，這邊修改 `README.md`，增加 11/30。

    ![](https://i.imgur.com/HOBj8K6.png)

4. 滑至底下，點擊 `Commit Changes`，此時就觸發了 Github Webhook 至 Jenkins 的作業。

    ![](https://i.imgur.com/mkjXwaz.png)

5. 回到 ngrok 的網址，就可以建置歷程多了一個 `#1`，並且綠色打勾表示成功，此外可以在點擊 `#1` 看詳細情況。

    <img src="https://i.imgur.com/nErfT0L.png" alt="Graph" width="55%"/>

6. `主控台輸出` > 可以看到有執行 `event from github`，就是先前在 Build 中指定觸發時所要執行的內容。

    ![](https://i.imgur.com/pQZJDad.png)

## 參考資料
[圖解Jenkins教學 - Jenkins & Github 整合教學 (webhook) 實際操作](https://www.youtube.com/watch?v=MmT5lYD5cy0&list=PLVVMQF8vWNCLNxRufOL4-L6Jh6NptawNh&index=1)

