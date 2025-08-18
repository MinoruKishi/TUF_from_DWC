# ToUnderstandFORTH using forth-DWC  
[ Click here for English version](README_ENG.md)   
  
## ChatGPTを使ってFORTH言語で書かれたゲームを組み立ててみる<br/>
ChatGPTを使ってIchigoJamの川下りゲーム（BASIC言語）をforth-DWCのプログラム（FORTH言語）にできるか調査する。詳細は以下の会話記録を参照してください。  
  
まだ現在進行中です。  
現段階はまずはChatGPTの問題解決の流れを理解して、私流のプログラミングを見つけるために参考になるアイデアを探しています。  
  
（参考）  
IchigoJam かわくだりゲーム改造法 ITってなに？フェスティバル プログラミング体験の復習（日本語のみ）  
https://fukuno.jig.jp/1924  
  
forth-dwcについてはFacebookのFORTH2020グループの会話の中で教えていただきました。  
https://github.com/CCurl/forth-dwc/tree/main  
  
---
  
# 会話記録：  
  
## ChatGPTとのforth-dwcの理解を深める  
  
[ まずはChatGPTがどの程度FORTH言語のことを理解しているか聞いてみた ](ChatGPT_Kawakudari_01.MD)  
  File:ChatGPT_Kawakudari_01  
  
  IchigoJam BASIC の「川下りゲーム」が好きなので、これをFORTH言語に置き換えたいと思った。そのために考えて置かなければならないことをChatGPT（4o）に聞いてみた。断られるかもしれないと思っていたが、かなり具体的で的確な回答をもらうことができた。
  
[ 私とChatGPT（4o）とで目的通りの開発ができるのか？ ](ChatGPT_Kawakudari_02b.MD)  
  File:ChatGPT_Kawakudari_02  
  
  この練習の目的は実際に起動するプログラムが開発できて、目的通りの動作が確認できることだった。そのことを私とChatGPT（4o）とがうまく連携して目的を達成できるのかを確認してみた。
  
  
[ forth-dwcのソースコードの理解を深めるためにいくつか質問した ](ChatGPT_Kawakudari_03.MD)  
  File:ChatGPT_Kawakudari_03  
  ここからはChatGPT（5）を使用した  
  
  問題なくFORTH-DWCで「川下りゲーム」の開発が可能とわかったので、それに取り掛かるためにFORTH-DWCのソースコードの理解を行った。ここで、ChatGPT（5）から「FORTH-DWCについては知りません」と言われるかもしれないと思っていたが、ChatGPT（5）の膨大なデータベースには関連する情報がすでに記録されていたようで、FORTH-DWCの開発者の作成した資料以外の情報も含めた詳細な説明を答えてくれた。正直驚いた。ないとは思うが、ChatGPT（5）に知らないことをごまかされても今の私には判断がつかない。これから理解を進めるに従って、私がChatGPT（5）に間違いを指摘できるようになるまで理解を深めたいと思っている（もっとも、今のペースで理解すると１００年くらい時間がかかりそうだけれども）。
  
---
  




