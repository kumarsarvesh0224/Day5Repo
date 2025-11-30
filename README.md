
using System;

class Program
{

    static void Main()
    {
        string[] inputArray = { "abc", "abcd", "efgh" };
        
        foreach (string word in GetLongestStrings(inputArray))
        {
            Console.WriteLine(word); 
        }

    }

    static string[] GetLongestStrings(string[] words)
    {

        int maxLength = 0;
        foreach (var word in words)
        {
            if (word.Length > maxLength)
                maxLength = word.Length;
        }


        int count = 0;
        foreach (var word in words)
        {
            if (word.Length == maxLength)
                count++;
        }


        string[] result = new string[count];
        int index = 0;
        foreach (var word in words)
        {
            if (word.Length == maxLength)
            {
                result[index] = word;
                index++;
            }
        }

        return result;
    }




}
