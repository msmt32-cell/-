import React, { useState, useEffect } from "react";

function App() {
  const [tab, setTab] = useState("home");
  const [isLoaded, setIsLoaded] = useState(false);

  const [inventory, setInventory] = useState({
    "1동": 0,
    "2동": 0,
    "3동": 0,
    "4동": 0,
    "5동": 0,
  });

  const [expenses, setExpenses] = useState([]);
  const [sales, setSales] = useState([]);
  const [memos, setMemos] = useState([]);

  const [newMemo, setNewMemo] = useState({ date: "", text: "" });
  const [newExpense, setNewExpense] = useState({ date: "", amount: "", description: "" });
  const [newSale, setNewSale] = useState({ date: "", amount: "", description: "" });

  // 불러오기 (먼저 실행)
  useEffect(() => {
    const data = JSON.parse(localStorage.getItem("farmData"));
    if (data) {
      setInventory(data.inventory || {});
      setExpenses(data.expenses || []);
      setSales(data.sales || []);
      setMemos(data.memos || []);
    }
    setIsLoaded(true);
  }, []);

  // 저장 (로드 후에만 실행)
  useEffect(() => {
    if (isLoaded) {
      localStorage.setItem(
        "farmData",
        JSON.stringify({ inventory, expenses, sales, memos })
      );
    }
  }, [inventory, expenses, sales, memos, isLoaded]);

  const totalInventory = Object.values(inventory).reduce((a, b) => a + b, 0);
  const totalExpense = expenses.reduce((a, b) => a + b.amount, 0);
  const totalSales = sales.reduce((a, b) => a + b.amount, 0);

  const handleAddMemo = () => {
    if (newMemo.date && newMemo.text) {
      setMemos([...memos, { ...newMemo, id: Date.now() }]);
      setNewMemo({ date: "", text: "" });
    } else {
      alert("날짜와 내용을 모두 입력해주세요.");
    }
  };

  const handleAddExpense = () => {
    if (newExpense.date && newExpense.amount) {
      setExpenses([
        ...expenses,
        { ...newExpense, amount: Number(newExpense.amount), id: Date.now() },
      ]);
      setNewExpense({ date: "", amount: "", description: "" });
    } else {
      alert("날짜와 금액을 입력해주세요.");
    }
  };

  const handleAddSale = () => {
    if (newSale.date && newSale.amount) {
      setSales([
        ...sales,
        { ...newSale, amount: Number(newSale.amount), id: Date.now() },
      ]);
      setNewSale({ date: "", amount: "", description: "" });
    } else {
      alert("날짜와 금액을 입력해주세요.");
    }
  };

  const handleDeleteMemo = (id) => {
    setMemos(memos.filter((m) => m.id !== id));
  };

  const handleDeleteExpense = (id) => {
    setExpenses(expenses.filter((e) => e.id !== id));
  };

  const handleDeleteSale = (id) => {
    setSales(sales.filter((s) => s.id !== id));
  };

  const sortedMemos = [...memos].sort((a, b) => new Date(b.date) - new Date(a.date));

  return (
    <div style={{ padding: 20, fontFamily: "Arial" }}>
      <h1>🦪 전복 치패 관리</h1>

      <div style={{ marginBottom: 20 }}>
        <button onClick={() => setTab("home")} style={{ marginRight: 10 }}>
          홈
        </button>
        <button onClick={() => setTab("inventory")} style={{ marginRight: 10 }}>
          개체수
        </button>
        <button onClick={() => setTab("money")} style={{ marginRight: 10 }}>
          수익
        </button>
        <button onClick={() => setTab("calendar")}>메모</button>
      </div>

      {tab === "home" && (
        <div style={{ border: "1px solid #ccc", padding: 15, borderRadius: 5 }}>
          <h3>📊 대시보드</h3>
          <p>총 개체수: <strong>{totalInventory}</strong>개</p>
          <p>총 매출: <strong>₩{totalSales.toLocaleString()}</strong></p>
          <p>총 비용: <strong>₩{totalExpense.toLocaleString()}</strong></p>
          <p style={{ color: totalSales - totalExpense >= 0 ? "green" : "red" }}>
            순이익: <strong>₩{(totalSales - totalExpense).toLocaleString()}</strong>
          </p>
        </div>
      )}

      {tab === "inventory" && (
        <div>
          <h2>개체수 관리</h2>
          {Object.keys(inventory).map((key) => (
            <div key={key} style={{ marginBottom: 10 }}>
              <label>
                {key}:
                <input
                  type="number"
                  min="0"
                  value={inventory[key]}
                  onChange={(e) =>
                    setInventory({ ...inventory, [key]: Math.max(0, Number(e.target.value)) })
                  }
                  style={{ marginLeft: 10, padding: 5 }}
                />
              </label>
            </div>
          ))}
        </div>
      )}

      {tab === "money" && (
        <div>
          <h2>💰 수익관리</h2>

          <div style={{ marginBottom: 20, border: "1px solid #ddd", padding: 10 }}>
            <h3>비용 추가</h3>
            <input
              type="date"
              value={newExpense.date}
              onChange={(e) => setNewExpense({ ...newExpense, date: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <input
              type="number"
              placeholder="금액"
              value={newExpense.amount}
              onChange={(e) => setNewExpense({ ...newExpense, amount: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <input
              type="text"
              placeholder="설명 (선택)"
              value={newExpense.description}
              onChange={(e) => setNewExpense({ ...newExpense, description: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <button onClick={handleAddExpense}>추가</button>
          </div>

          <div style={{ marginBottom: 20, border: "1px solid #ddd", padding: 10 }}>
            <h3>매출 추가</h3>
            <input
              type="date"
              value={newSale.date}
              onChange={(e) => setNewSale({ ...newSale, date: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <input
              type="number"
              placeholder="금액"
              value={newSale.amount}
              onChange={(e) => setNewSale({ ...newSale, amount: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <input
              type="text"
              placeholder="설명 (선택)"
              value={newSale.description}
              onChange={(e) => setNewSale({ ...newSale, description: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <button onClick={handleAddSale}>추가</button>
          </div>

          <div style={{ marginBottom: 20 }}>
            <h3>비용 목록</h3>
            {expenses.length === 0 ? (
              <p>비용이 없습니다.</p>
            ) : (
              expenses.map((e) => (
                <div key={e.id} style={{ display: "flex", justifyContent: "space-between", padding: 5, borderBottom: "1px solid #eee" }}>
                  <span>{e.date} - {e.description || "비용"}: ₩{e.amount.toLocaleString()}</span>
                  <button onClick={() => handleDeleteExpense(e.id)}>삭제</button>
                </div>
              ))
            )}
          </div>

          <div style={{ marginBottom: 20 }}>
            <h3>매출 목록</h3>
            {sales.length === 0 ? (
              <p>매출이 없습니다.</p>
            ) : (
              sales.map((s) => (
                <div key={s.id} style={{ display: "flex", justifyContent: "space-between", padding: 5, borderBottom: "1px solid #eee" }}>
                  <span>{s.date} - {s.description || "매출"}: ₩{s.amount.toLocaleString()}</span>
                  <button onClick={() => handleDeleteSale(s.id)}>삭제</button>
                </div>
              ))
            )}
          </div>

          <h3>📈 요약</h3>
          <p>총매출: <strong>₩{totalSales.toLocaleString()}</strong></p>
          <p>총비용: <strong>₩{totalExpense.toLocaleString()}</strong></p>
          <p style={{ color: totalSales - totalExpense >= 0 ? "green" : "red" }}>
            순이익: <strong>₩{(totalSales - totalExpense).toLocaleString()}</strong>
          </p>
        </div>
      )}

      {tab === "calendar" && (
        <div>
          <h2>📝 메모</h2>

          <div style={{ marginBottom: 20, border: "1px solid #ddd", padding: 10 }}>
            <input
              type="date"
              value={newMemo.date}
              onChange={(e) => setNewMemo({ ...newMemo, date: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <input
              placeholder="내용"
              value={newMemo.text}
              onChange={(e) => setNewMemo({ ...newMemo, text: e.target.value })}
              style={{ marginRight: 10 }}
            />
            <button onClick={handleAddMemo}>저장</button>
          </div>

          <h3>메모 목록</h3>
          {memos.length === 0 ? (
            <p>메모가 없습니다.</p>
          ) : (
            sortedMemos.map((m) => (
              <div key={m.id} style={{ display: "flex", justifyContent: "space-between", padding: 10, border: "1px solid #eee", marginBottom: 5, borderRadius: 5 }}>
                <div>
                  <strong>{m.date}</strong> - {m.text}
                </div>
                <button onClick={() => handleDeleteMemo(m.id)}>삭제</button>
              </div>
            ))
          )}
        </div>
      )}
    </div>
  );
}

export default App;
