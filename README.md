# Labor 13

## Feladatok

<img src="https://i.ibb.co/TMRp6W9/blockbuster.png" width="300px" align="left"> 
<br />
<p>
  1. Készítsünk egy <a href="https://elgoog.im/breakout">Breakout játékot</a>. Egy 600x400-as ablakban jelenítsünk meg 7x5 téglát, amelyet ki tudunk majd törni. Az órán implementált játékot (letölthető <a href="https://drive.google.com/file/d/1DhPTbESJCP2JnE6bqzrZyLer1T-Olvfi/view?usp=sharing">innen</a>) bővítsük ki a következőképp:
  <ul>
    <li> készítsünk <i>Game Over</i> feliratot játék végén (ehhez használhatjuk akár a <code>QGraphicsScene::addText</code> [https://doc.qt.io/qt-5/qgraphicsscene.html#addText] metódusát), illetve implementáljunk egy újrakezdhetőségi mechanizmust (legyen az visszaszámolós, vagy menüből kiválaszható opció)</li>
    <li> adjunk lehetőséget a menüből átállítani a téglák (és/vagy golyó, csúszka) színét</li>
    <li> adjunk lehetőséget nehezebb játékmód kiválasztására, ahol: 
         <ol>
           <li> a téglák kitöréséhez megemeljük az ütközések számát, illetve a golyó sebességét</li>
           <li> illetve egy másikat, amely során növeljük a golyó sebességét a kiütött téglák függvényében</li>
         </ol>
     </li>
    <li> a status bar részen jelenítsük meg hány tégla maradt, illetve hányat törtünk ki</li>
  <ul>
</p>
