> ๐ ์ฒซ ํผ๋๋ฐฑ์ ๋ฐ๋ค.

๋ฐ์ฌ์ฑ๋์ TDD๊ฐ์ ch1 ๋ฉ์ธ ์ค์ต์ธ '์ซ์ ์ผ๊ตฌ๊ฒ์ ๊ตฌํ'์ ํ๊ณ , PR์ ์์ฒญํ์์๋ค.

๋ฐ์์์ง๋ง ์๊ฐ ๋  ๋ ํผ๋๋ฐฑ ์ฃผ์๊ฒ ๋ค๋ ์น์ ํ ๋ฆฌ๋ทฐ์ด๋..

์ฒซ ํผ๋๋ฐฑ์ ๋ฐ์๊ณ , ๋ค์ ์ด๋ ต๊ฒ ์ง๋ ๊ฒฝํฅ์ด ์๋ค๊ณ  ํ์จ๋ค.

๊ณ ๋ฏผ์ ํ๋ค๊ฐ, ์ซ์ ์ผ๊ตฌ๊ฒ์์ ์  ๋จ๊ณ์ธ ๋ฌธ์์ด ๊ณ์ฐ๊ธฐ๋ฅผ ๋จผ์  ํด๋ณด๊ธฐ๋ก ํ๋ค.

ํน์  ์ต๊ด์ ์ค์ด๋ ค๋ฉด ์๋์ ์ผ๋ก ์ฌ์ด ๊ฒ์์๋ถํฐ ๊ณ ์ณ๋๊ฐ์ผ ํ๋ค๊ณ  ์๊ฐํ๊ธฐ ๋๋ฌธ์ด๋ค.

์ถ๊ฐ์ ์ผ๋ก ์ผ๊ตฌ๊ฒ์์ ํผ๋๋ฐฑ๋ค ์ค, ์ ์ฉํ  ์ ์๋ ๊ฒ์ ์ ์ฉํ๋ ค๊ณ  ์๊ฐ์ด ๋ค์๋ค.

- ๋ฌธ์์ด ํ์คํธ์ ์ ์ฉํ  ์ ์๋ ํผ๋๋ฐฑ๋ค ๋์ด
1. EOF

2. ์ปค๋ฐ ์ปจ๋ฒค์

3. ์ฝ๋ฉ ์ปจ๋ฒค์(IDEํ์ฉ)

4. toString์ ํ๋ฉด ์๊ตฌ์ฌํญ์ ์ํด ์ฌ์ฉํ์ง ๋ง ๊ฒ
-> ์ฃผ๋ก ์ด๋์ ์ฌ์ฉ๋ ๊น? ๋ก๊น?

5. setter ๋์  ๋ช์์  ์ด๊ธฐํ / ์์ฑ์ / ํ ๋ฉ์๋

6. ๋งค์ง๋๋ฒ(์์๊ฐ?)

7. ํต์ฌ ๋ก์ง์ ๊ตฌํํ๋ ์ฝ๋์ UI๋ฅผ ๋ด๋นํ๋ ๋ก์ง์ ๊ตฌ๋ถํ๋ค.

![](https://images.velog.io/images/sonchanwoo/post/b62b05f6-85a8-49ea-8212-edd155fe234d/20220120_171416.png)

<br/>

8. ์๋ฃ๊ตฌ์กฐ์ ์ด๋ฆ์ ๊ทธ๋๋ก ์ฌ์ฉx(list), set ์ผ๋ก ๋ฐ๋๋ฉด?

9. ์ ํธํด๋์ค๋ ๋ค๋ฅธ ์๋น์ค์์๋ ์ด์ฉ๊ฐ๋ฅํ ๊น๋ฅผ ๊ณ ๋ฏผํ๋ค.

10. @DisplayName()์ ๊ทน ์ฌ์ฉ

11. reflection
(1) reflection ๊ฐ๋์ฑ โ, ์ด์ฉ ์ ์๋ ๊ฒฝ์ฐ ์์ง๋ง, ๊ทธ๋ฐ ์ํฉ ์ ๋ง๋ค๊ธฐ
(2) ํ๋์ public๋ฉ์๋๊ฐ ๋ง์ ์ฑ์(private's')์ ๊ฐ์ง๊ณ  ์์ ์ ์๋ ์ ํธ
-> private method๋ ํ์คํธ๋ฅผ ํ์ง ์๋ ๊ฒ์ธ๊ฐ?(public์ผ๋ก ํ์คํธ ํ๊ณ  ํ์คํธ๋ฅผ ์ง์ฐ๊ฑฐ๋ ์ฃผ์์ฒ๋ฆฌ ํ๋ ๊ฑด๊ฐ?)

12. ๋ฌด์ธ๊ฐ ๋ฐ๋ณต์ ์ธ ์์์ ํ ๋ ์ด์ฉํ  ๋งํ ๊ฒ์ด ์๋์ง ๊ณ ๋ฏผ(Arrays, ๊ฐ๋ณ์ธ์ํ๋ผ๋ฏธํฐ ๋ฑ)


<br/>

> ๐ ์ข ๋ ์์๋ณผ ์๊ตฌ์ฌํญ๋ค

1. EOF ์ค์  - intellij ์ค์  ํฌ์คํ์ ๋ฐ์ ์๋ฃ.(์ดํ ์ฐธ๊ณ ๋งํฌ)
-> https://minz.dev/19
-> https://stackoverflow.com/questions/729692/why-should-text-files-end-with-a-newline

<br/>

2. ์ปค๋ฐ ์ปจ๋ฒค์
  (-) ์ง๊ธ ํ  ๋ด์ฉ์ด ์๋ ๋ฏ ํ๋.. ๊ธฐ๋ณธ์ ์ธ ๊ฒ ํฌ์คํ ํ์๋ค.

<br/>

3. ์ฝ๋ฉ ์ปจ๋ฒค์
  (-) ํฌ์คํ ๋ฐ ์ค์  ํฌ์คํ ๋ฐ์ ์๋ฃ
  
<br/>

6. ๋งค์ง ๋๋ฒ ๊ธ์ง : ์์๊ฐ ํฌ์ฅํ๋ผ
 (-) ๊ฐ๋์ฑ์ ์ํ์ฌ ์์ ๊ฐ์ ํฌ์ฅํด์ผ ํ๋ค.
 (-) ๋ผ์๊ฑฐ๋ฆฌ ์ค ํ๋์ด๋ฉฐ, ๊ฐํน ๋งค์ง๋๋ฒ๋ฅผ ์ฌ์ฉํ๋ ๊ฒ ๊ฐ๋์ฑ์ด ์ข์์ง๋ ๊ฒฝ์ฐ๊ฐ ์๋ค๊ณ  ํ๋ค. ๋ค์์ ์๋ ๋งํฌ๋ฅผ ๋ณด์!
-> https://stackoverflow.com/questions/47882/what-is-a-magic-number-and-why-is-it-bad

<br/>

> ๐ ๋ค์์ ํ  ์ผ
- ์ ์ฌํญ๋ค์ ๋ชจ๋ ์ ์ฉํ์ฌ, ๋ฌธ์์ด ๊ณ์ฐ๊ธฐ๋ฅผ ํด๋ณผ ์๊ฐ์ด๋ค.