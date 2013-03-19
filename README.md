ssql
====

simple scalding query language

ssql is an unforgiving vso ( verb, subject, object ) dialect of Scalding.

Use case: Nonprogrammers, Managers, Non-scala-programmers who are interested in Scalding

Usage: scala ssql yourscript
Result: yourscript.scala created!
Then ? scald.rb --local yourscript.scala

Legit ssql verbs: open, save, rows, columns, column

Example Script: Type this in a text file named "somescript"
---------
open employees file1.txt
columns 3
column 1 name text
column 2 age number
column 3 income decimal
rows select age > 20
rows select age < 27
column add newincome income*1.1 + (age-20)*100
column remove age
column remove income
save file2.txt
----------

scala ssql somescript
somescript.scala created!

----somescript.scala -----
import com.twitter.scalding._
class somescript(args : Args) extends Job(args){
  val employees = 
    TextLine("file1.txt")
    .read
    .mapTo('line -> ('name,'age,'income) ){
    line:String =>
    val res = line.split(" ").map( _.trim).filter(_.length > 0)
    val name:String = res(0).toString
    val age:Int = res(1).toInt
    val income:Double = res(2).toDouble
    (name,age,income)
  }.filter('age) { 
    age:Int => 
    (age > 20)
  }.filter('age) { 
    age:Int => 
    (age < 27)
  }.map(('income,'age) -> ('newincome)) {
    columns:(Double,Int)=> 
    val (income,age) = columns
    income*1.1 + (age-20)*100
  }.discard('age)
    .discard('income)
    .write(Tsv("file2.txt"))
}
-------------------

Given file1.txt
---
fred 25 10000
sam 25 150000
oscar   39  200000
baker 27  234567
pascal 24 123456
ford 22 55000
haskell 56 45466
----

scald.rb --local somescript.scala

Result: file2.txt
---
fred  11500.0
sam	165500.0
pascal	136201.6
ford	60700.0
---




