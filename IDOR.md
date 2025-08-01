IDOR - Insecure Direct Object References

# חשוב !
* זה פגיעות שמזכירה את LFI אבל הם שונות במהות:
	* פגיעות [[LFI]] מנצלת **את מנגנון טעינת הקבצים של המערכת** לחשוף קבצים שלא אמורים לראות
	* פגיעות [[IDOR]] מנצלת **את מנגנון אימות משתמשים לא נכון** שמאפשר לנו לדלג מהמשתמש שלנו אל משתמש אחר או להחליף אותו, זה bypass למנגנון אימות במערכת
# הסבר
* סוג של פגיעות אבטחה כאשר אתר מאפשר למשתמש לגשת ולשנות משאבים שלא היה אמורה להיות גישה אליהם.
* לדוגמא אם אני באתר שמציג נתונים של משתמש ואני רואה בURL:
	* משתנה `id=1` - כמו ת"ז
```
https://example.com/user?id=1
https://insecure-website.com/customer_account?customer_number=132355
```
* במצב כזה פגיעות תקרה אם אני משנה את id למשל אל 2 ואז אני רואה פרטים של משתמש אחר

# אינדיקציה
1) לחפש פרמטרים רלוונטים - id הכי נפוץ אבל כל משתנה שיכול להוות מזהה כמו ת"ז, מספר אישי, מספר לקוח וכן הלאה
2) מניפולציה של הפרמטרים - לנסות לשנות ערכים
	1) לפעמים השינוי יכול להיות עקבי - מזהים בסדר עולה 1,2,3 וכן הלאה
	2) השינוי אקראי - אין סדר בין המזהים ואז יותר קשה לדלג בין משתמשים - לא הכי אפשרי אז פחות סביר
3) קבלת גישה לא מורשית

# ניצול
* נניח זיהינו שיש פרמטר שחושדים שפגיע
```
http://127.0.0.1/profile.php?user=34
```
* במקום לנסות ידנית מספרים, כמו שאמרנו לא בהכרח יהיו סמוכים ניתן להשתמש בintruder בתוך burp
* בגדול אפשר להגיד לburp שישים בid כל מספר מ1 עד 10000 למשל
	* לשאול את הAI איך לעשות את זה ויש גם מדריכים באינטרנט
* **דגש חשוב** - burp לא הכי מהיר, אם החלטתם שאתם מנסים ככה זה יכול לקחת כמה דקות, אז אפשר להתקדם לתרגיל אחר בינתיים
* בתוך burp נזהה שיש בקשות באורך אחר וזה אומר שכנראה הערך שהוא שם במשתנה השפיע על התגובה
* בדוגמא כאן כנראה id=14 ייתן לנו גישה למשתמש אחר
* <img width="521" height="118" alt="Pasted image 20250723012652" src="https://github.com/user-attachments/assets/0d7e83ec-ac9e-471a-915f-a9fe2f62f348" />


# הערות
* אובייקט יכול להיות בכמה מקומות:
1) בURL - הכי נפוץ, כמשתנה
2) גוף הבקשה - תחת משתנה מסוים
3) בתוך cookie - יותר נדיר אבל זה יכול להיות
	* בסוף cookie הוא מזהה של סשן, אז אם האובייקט הוא עבור משתמש, ייתכן שתהיה עוגייה של אובייקט משתמש![Uploading Pasted image 20250723012652.png…]()
