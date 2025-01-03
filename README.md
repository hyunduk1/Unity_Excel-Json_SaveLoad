# Unity_Excel-Json_SaveLoad


    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System.Text.RegularExpressions;
    using System.IO;
    public class CDataMng : MonoBehaviour
    {
        [System.Serializable]
        public class SaveData
        {
            public int playerScore;
            public Vector3 EditPosition;
        }

    private static readonly string SPLIT_RE = @",(?=(?:[^""]*""[^""]*"")*(?![^""]*""))";
    private static readonly string LINE_SPLIT_RE = @"\r\n|\n\r|\n|\r";
    private static readonly char[] TRIM_CHARS = { '\"' };

    private static CDataMng _instance;
    public static CDataMng Instance { get { return _instance; } }

    private string filePath;

    private void Awake()
    {
        if (_instance == null)
        {
            _instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
        filePath = Path.Combine(Application.persistentDataPath, "saveData.json");
    }

    // Start is called before the first frame update
    void Start()
    {
        //저장
        //ModifyCSV("Position", CRenderCameraMng.Instance._goOutPutObject.transform.GetChild(4).GetComponent<RectTransform>().anchoredPosition.x.ToString(), 0, "4_Object");

        //로드
        //CDataMng.Instance.Read("Position")[0]["3_Object"]

    }

    // Update is called once per frame
    void Update()
    {
    }

    //---------------------Excel Save Load--------------------------------
    public List<Dictionary<string, object>> Read(string file)
    {
        var list = new List<Dictionary<string, object>>();
        string filePath = Path.Combine(Application.streamingAssetsPath, file + ".csv");

        if (!File.Exists(filePath))
        {
            Debug.LogError($"CSV file not found: {filePath}");
            return list;
        }

        string[] lines = File.ReadAllLines(filePath);

        if (lines.Length <= 1)
            return list;

        var header = Regex.Split(lines[0], SPLIT_RE);
        for (var i = 1; i < lines.Length; i++)
        {
            var values = Regex.Split(lines[i], SPLIT_RE);
            if (values.Length == 0 || values[0] == "")
                continue;

            var entry = new Dictionary<string, object>();
            for (var j = 0; j < header.Length && j < values.Length; j++)
            {
                string value = values[j];
                value = value.TrimStart(TRIM_CHARS).TrimEnd(TRIM_CHARS).Replace("\\", "");
                object finalvalue = value;
                if (int.TryParse(value, out int n))
                {
                    finalvalue = n;
                }
                else if (float.TryParse(value, out float f))
                {
                    finalvalue = f;
                }
                entry[header[j]] = finalvalue;
            }
            list.Add(entry);
        }
        return list;
    }

    /// <summary>
    /// file : csv 파일이름 , newvalue : 수정할 데이터 값, rowIndex : 몇번째 행 인지, conditionkey : 어떤 열의 행을 바꿀건지
    /// </summary>
    /// <param name="file"> 파일 네임 </param>
    /// <param name="newValue"> 수정할 데이터 값 </param>
    /// <param name="rowIndex"> 몇번째 행인지</param>
    /// <param name="conditionKey"> 어떤 열의 행을 바꿀건지 </param>
    public void ModifyCSV(string file, string newValue, int rowIndex, string conditionKey)
    {
        string filePath = Path.Combine(Application.streamingAssetsPath, file + ".csv");
        rowIndex = rowIndex + 1;
        if (!File.Exists(filePath))
        {
            Debug.LogError($"CSV file not found: {filePath}");
            return;
        }

        string[] lines = File.ReadAllLines(filePath);
        var modifiedLines = new List<string>();

        var header = Regex.Split(lines[0], SPLIT_RE);
        modifiedLines.Add(lines[0]);

        for (int i = 1; i < lines.Length; i++)
        {
            string line = lines[i];
            string[] values = Regex.Split(line, SPLIT_RE);

            if (i == rowIndex)
            {
                for (int j = 0; j < header.Length; j++)
                {
                    if (header[j] == conditionKey)
                    {
                        values[j] = newValue;
                    }
                }
            }

            modifiedLines.Add(string.Join(",", values));
        }

        File.WriteAllLines(filePath, modifiedLines.ToArray());
    }

    //----------------------json Save Load--------------------------------
    public void Save(SaveData data)
    {
        string json = JsonUtility.ToJson(data, true);
        File.WriteAllText(filePath, json);
        Debug.Log($"Data saved to {filePath}");
    }
    public SaveData Load()
    {
        if (File.Exists(filePath))
        {
            string json = File.ReadAllText(filePath);
            SaveData data = JsonUtility.FromJson<SaveData>(json);
            Debug.Log($"Data loaded from {filePath}");
            return data;
        }
        else
        {
            Debug.LogWarning("Save file not found!");
            return null;
        }
    }
}
