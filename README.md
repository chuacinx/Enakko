import { useState, useMemo } from "react";
import { Minus, Plus, ClipboardList, Scale } from "lucide-react";

const UNITS = [
  { key: "utuh", label: "Ekor Utuh", sub: "1 ekor", value: 1 },
  { key: "setengah", label: "Setengah", sub: "1/2 ekor", value: 0.5 },
  { key: "seperempat", label: "Seperempat", sub: "1/4 ekor", value: 0.25 },
];

function fmt(n) {
  const r = Math.round(n * 100) / 100;
  return Number.isInteger(r) ? r.toString() : r.toFixed(2).replace(/0+$/, "").replace(/\.$/, "");
}

function Counter({ label, sub, value, onChange }) {
  return (
    <div className="flex items-center justify-between py-4 border-b border-[#e4ddd0] last:border-none">
      <div>
        <p className="font-semibold text-[#2b2620] tracking-tight">{label}</p>
        <p className="text-xs text-[#8a7f6c] font-mono">{sub}</p>
      </div>
      <div className="flex items-center gap-3">
        <button
          onClick={() => onChange(Math.max(0, value - 1))}
          className="w-8 h-8 rounded-full border border-[#d8cfbc] flex items-center justify-center text-[#6b6252] hover:bg-[#efe9dc] active:scale-95 transition"
          aria-label={`Kurangi ${label}`}
        >
          <Minus size={14} />
        </button>
        <input
          type="number"
          min="0"
          value={value}
          onChange={(e) => onChange(Math.max(0, Number(e.target.value) || 0))}
          className="w-14 text-center font-mono text-lg font-semibold bg-transparent focus:outline-none focus:ring-2 focus:ring-[#3f6b4a] rounded"
        />
        <button
          onClick={() => onChange(value + 1)}
          className="w-8 h-8 rounded-full bg-[#3f6b4a] text-white flex items-center justify-center hover:bg-[#345a3e] active:scale-95 transition"
          aria-label={`Tambah ${label}`}
        >
          <Plus size={14} />
        </button>
      </div>
    </div>
  );
}

export default function KalkulatorAyam() {
  const [counts, setCounts] = useState({ utuh: 0, setengah: 0, seperempat: 0 });
  const [stok, setStok] = useState(0);

  const totalPakai = useMemo(
    () => UNITS.reduce((sum, u) => sum + counts[u.key] * u.value, 0),
    [counts]
  );

  const selisih = useMemo(() => stok - totalPakai, [stok, totalPakai]);

  let status = { label: "Pas / cocok", color: "#3f6b4a", bg: "#e9f2ea" };
  if (selisih > 0.001) status = { label: "Sisa stok", color: "#a3781f", bg: "#f7eed8" };
  if (selisih < -0.001) status = { label: "Kurang / lebih pakai", color: "#a8402f", bg: "#f8e6e0" };

  return (
    <div className="min-h-screen bg-[#f7f3e9] flex justify-center px-4 py-10">
      <div className="w-full max-w-md space-y-5">
        <header className="mb-2">
          <p className="text-xs uppercase tracking-[0.2em] text-[#8a7f6c] font-mono mb-1">Cinx Trade · Kios</p>
          <h1 className="text-3xl font-bold text-[#2b2620] tracking-tight">Kalkulator Ayam</h1>
          <p className="text-sm text-[#8a7f6c] mt-1">Hitung pemakaian harian & cocokkan dengan stok.</p>
        </header>

        {/* Kartu 1: Pemakaian */}
        <div className="bg-white rounded-2xl border border-[#e4ddd0] shadow-sm overflow-hidden">
          <div className="flex items-center gap-2 px-5 pt-5">
            <ClipboardList size={16} className="text-[#3f6b4a]" />
            <h2 className="text-sm font-semibold uppercase tracking-wide text-[#2b2620]">Pemakaian Hari Ini</h2>
          </div>
          <div className="px-5 mt-2">
            {UNITS.map((u) => (
              <Counter
                key={u.key}
                label={u.label}
                sub={u.sub}
                value={counts[u.key]}
                onChange={(v) => setCounts((c) => ({ ...c, [u.key]: v }))}
              />
            ))}
          </div>
          <div className="flex items-center justify-between px-5 py-4 bg-[#2b2620] mt-2">
            <span className="text-[#d8cfbc] text-sm font-medium">Total Pemakaian</span>
            <span className="text-white font-mono text-2xl font-bold">{fmt(totalPakai)} <span className="text-sm font-normal text-[#d8cfbc]">ekor</span></span>
          </div>
        </div>

        {/* Kartu 2: Pembanding stok */}
        <div className="bg-white rounded-2xl border border-[#e4ddd0] shadow-sm overflow-hidden">
          <div className="flex items-center gap-2 px-5 pt-5">
            <Scale size={16} className="text-[#3f6b4a]" />
            <h2 className="text-sm font-semibold uppercase tracking-wide text-[#2b2620]">Pembanding Stok Aktual</h2>
          </div>

          <div className="flex items-center justify-between px-5 py-4 border-b border-[#e4ddd0] mt-2">
            <div>
              <p className="font-semibold text-[#2b2620]">Stok Aktual</p>
              <p className="text-xs text-[#8a7f6c] font-mono">hasil hitung fisik (ekor)</p>
            </div>
            <input
              type="number"
              min="0"
              step="0.25"
              value={stok}
              onChange={(e) => setStok(Math.max(0, Number(e.target.value) || 0))}
              className="w-20 text-center font-mono text-lg font-semibold border border-[#d8cfbc] rounded-lg py-1 focus:outline-none focus:ring-2 focus:ring-[#3f6b4a]"
            />
          </div>

          <div className="px-5 py-4 space-y-2">
            <div className="flex justify-between text-sm text-[#6b6252]">
              <span>Total pemakaian</span>
              <span className="font-mono">{fmt(totalPakai)} ekor</span>
            </div>
            <div className="flex justify-between text-sm text-[#6b6252]">
              <span>Stok aktual</span>
              <span className="font-mono">{fmt(stok)} ekor</span>
            </div>
          </div>

          <div
            className="mx-5 mb-5 rounded-xl px-4 py-3 flex items-center justify-between"
            style={{ backgroundColor: status.bg }}
          >
            <span className="text-sm font-medium" style={{ color: status.color }}>{status.label}</span>
            <span className="font-mono text-lg font-bold" style={{ color: status.color }}>
              {selisih > 0 ? "+" : ""}{fmt(selisih)} ekor
            </span>
          </div>
        </div>

        <p className="text-center text-xs text-[#a89d87] font-mono">selisih + = sisa · selisih − = kurang</p>
      </div>
    </div>
  );
}

