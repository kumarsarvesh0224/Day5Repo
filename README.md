using ArrayStarAdditionApp.Domain;


namespace ArrayStarAdditionApp
{

    class Program
    {

        static void Main()
        {

            string[] starArray = { "Gunnu", "Sanya", "Sarvesh" };
            foreach (var item in StarArrayExtended(starArray))

            {
                Console.WriteLine(item);
            }
        }


        static string[] StarArrayExtended(string[] words)
        {

            string[] result = new string[words.Length + 2];

            result[0] = "***";

            for (int i = 0; i < words.Length; i++)
            {
                result[i + 1] = $"*{words[i]}*";
            }
            result[result.Length - 1] = "***";

            return result;
        }


    }
}


