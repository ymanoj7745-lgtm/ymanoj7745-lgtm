import { useState, useEffect, useRef } from "react";

const COLORS = {
  bg: "#080C10",
  surface: "#0D1117",
  card: "#111820",
  border: "#1C2733",
  accent: "#F0A500",
  accentGlow: "#F0A50033",
  green: "#2ECC71",
  blue: "#4FC3F7",
  red: "#E74C3C",
  purple: "#A78BFA",
  muted: "#4A5568",
  text: "#CDD9E5",
  textDim: "#768390",
};

const REPOS = [
  { name: "neural-core", lang: "Python", stars: 4821, forks: 312, issues: 27, commits: 1834, color: "#4FC3F7", trend: [12,18,9,24,15,30,22,28,35,20,42,38] },
  { name: "phantom-ui", lang: "TypeScript", stars: 3109, forks: 241, issues: 14, commits: 2291, color: "#A78BFA", trend: [5,12,8,19,22,14,28,25,31,20,35,40] },
  { name: "storm-engine", lang: "Rust", stars: 2047, forks: 178, issues: 43, commits: 876, color: "#F0A500", trend: [8,6,14,10,18,12,22,16,20,25,18,30] },
  { name: "skylink", lang: "Go", stars: 1563, forks: 99, issues: 8, commits: 1120, color: "#2ECC71", trend: [3,7,5,10,8,12,9,15,11,18,14,22] },
  { name: "void-db", lang: "C++", stars: 987, forks: 64, issues: 22, commits: 567, color: "#E74C3C", trend: [2,5,3,8,6,10,7,12,9,11,8,14] },
];

const ACTIVITY = [
  { day: "Mon", commits: 14, prs: 3, reviews: 7 },
  { day: "Tue", commits: 22, prs: 5, reviews: 12 },
  { day: "Wed", commits: 8, prs: 2, reviews: 4 },
  { day: "Thu", commits: 31, prs: 7, reviews: 18 },
  { day: "Fri", commits: 19, prs: 4, reviews: 9 },
  { day: "Sat", commits: 6, prs: 1, reviews: 3 },
  { day: "Sun", commits: 11, prs: 2, reviews: 6 },
];

const LANGS = [
  { name: "Python", pct: 38, color: "#4FC3F7" },
  { name: "TypeScript", pct: 27, color: "#A78BFA" },
  { name: "Rust", pct: 18, color: "#F0A500" },
  { name: "Go", pct: 11, color: "#2ECC71" },
  { name: "C++", pct: 6, color: "#E74C3C" },
];

const FEED = [
  { type: "push", repo: "neural-core", msg: "feat: add transformer attention layer", time: "2m ago", avatar: "🧠" },
  { type: "pr", repo: "phantom-ui", msg: "Merge PR #142: Dark mode tokens", time: "18m ago", avatar: "🎨" },
  { type: "issue", repo: "storm-engine", msg: "Bug: Vulkan pipeline race condition", time: "1h ago", avatar: "⚡" },
  { type: "star", repo: "neural-core", msg: "4 new stars from @devhunt", time: "2h ago", avatar: "⭐" },
  { type: "push", repo: "skylink", msg: "refactor: optimize gRPC pool handling", time: "3h ago", avatar: "🌐" },
  { type: "release", repo: "void-db", msg: "v2.4.1 — hotfix: NULL pointer deref", time: "5h ago", avatar: "🚀" },
];

function SparkLine({ data, color, width = 120, height = 36 }) {
  const max = Math.max(...data);
  const points = data.map((v, i) => {
    const x = (i / (data.length - 1)) * width;
    const y = height - (v / max) * (height - 4) - 2;
    return `${x},${y}`;
  }).join(" ");
  const fill = data.map((v, i) => {
    const x = (i / (data.length - 1)) * width;
    const y = height - (v / max) * (height - 4) - 2;
    return `${x},${y}`;
  });
  const areaPoints = `0,${height} ${fill.join(" ")} ${width},${height}`;

  return (
    <svg width={width} height={height} style={{ overflow: "visible" }}>
      <defs>
        <linearGradient id={`sg-${color.replace("#","")}`} x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stopColor={color} stopOpacity="0.3" />
          <stop offset="100%" stopColor={color} stopOpacity="0" />
        </linearGradient>
      </defs>
      <polygon points={areaPoints} fill={`url(#sg-${color.replace("#","")})`} />
      <polyline points={points} fill="none" stroke={color} strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round" />
      {data.map((v, i) => {
        const x = (i / (data.length - 1)) * width;
        const y = height - (v / max) * (height - 4) - 2;
        return i === data.length - 1 ? (
          <circle key={i} cx={x} cy={y} r="3" fill={color} />
        ) : null;
      })}
    </svg>
  );
}

function ContribGrid() {
  const weeks = 26;
  const days = 7;
  const grid = Array.from({ length: weeks }, () =>
    Array.from({ length: days }, () => Math.random() < 0.6 ? Math.floor(Math.random() * 5) : 0)
  );
  const levels = ["#1C2733", "#1B4332", "#276749", "#2ECC71", "#52E090"];
  return (
    <div style={{ display: "flex", gap: 3 }}>
      {grid.map((week, wi) => (
        <div key={wi} style={{ display: "flex", flexDirection: "column", gap: 3 }}>
          {week.map((val, di) => (
            <div key={di} style={{
              width: 10, height: 10, borderRadius: 2,
              background: levels[val],
              transition: "transform 0.1s",
            }} />
          ))}
        </div>
      ))}
    </div>
  );
}

function BarChart({ data }) {
  const maxC = Math.max(...data.map(d => d.commits));
  return (
    <div style={{ display: "flex", alignItems: "flex-end", gap: 6, height: 80 }}>
      {data.map((d, i) => (
        <div key={i} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
          <div style={{ width: "100%", display: "flex", flexDirection: "column", gap: 2, justifyContent: "flex-end", height: 64 }}>
            <div style={{ width: "100%", height: (d.reviews / maxC) * 60, background: COLORS.purple, borderRadius: "3px 3px 0 0", opacity: 0.6 }} />
            <div style={{ width: "100%", height: (d.prs / maxC) * 60, background: COLORS.blue, borderRadius: "3px 3px 0 0", opacity: 0.8 }} />
            <div style={{ width: "100%", height: (d.commits / maxC) * 60, background: COLORS.accent, borderRadius: "3px 3px 0 0" }} />
          </div>
          <span style={{ fontSize: 10, color: COLORS.textDim, fontFamily: "monospace" }}>{d.day}</span>
        </div>
      ))}
    </div>
  );
}

function DonutChart({ langs }) {
  const total = langs.reduce((s, l) => s + l.pct, 0);
  let cumulative = 0;
  const r = 52, cx = 65, cy = 65, stroke = 14;
  const circumference = 2 * Math.PI * r;
  const segments = langs.map(l => {
    const offset = cumulative;
    cumulative += l.pct / total;
    return { ...l, offset, length: l.pct / total };
  });

  return (
    <div style={{ display: "flex", alignItems: "center", gap: 20 }}>
      <svg width={130} height={130}>
        <defs>
          <filter id="glow">
            <feGaussianBlur stdDeviation="2" result="blur" />
            <feMerge><feMergeNode in="blur" /><feMergeNode in="SourceGraphic" /></feMerge>
          </filter>
        </defs>
        <circle cx={cx} cy={cy} r={r} fill="none" stroke={COLORS.border} strokeWidth={stroke} />
        {segments.map((seg, i) => (
          <circle key={i} cx={cx} cy={cy} r={r}
            fill="none" stroke={seg.color} strokeWidth={stroke}
            strokeDasharray={`${seg.length * circumference} ${circumference}`}
            strokeDashoffset={-seg.offset * circumference}
            strokeLinecap="butt"
            transform={`rotate(-90 ${cx} ${cy})`}
            style={{ filter: `drop-shadow(0 0 4px ${seg.color}80)` }}
          />
        ))}
        <text x={cx} y={cy - 6} textAnchor="middle" fill={COLORS.text} fontSize="18" fontWeight="bold" fontFamily="monospace">12.4k</text>
        <text x={cx} y={cy + 12} textAnchor="middle" fill={COLORS.textDim} fontSize="9" fontFamily="monospace">total stars</text>
      </svg>
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {langs.map((l, i) => (
          <div key={i} style={{ display: "flex", alignItems: "center", gap: 8 }}>
            <div style={{ width: 8, height: 8, borderRadius: "50%", background: l.color, boxShadow: `0 0 6px ${l.color}` }} />
            <span style={{ fontSize: 11, color: COLORS.textDim, fontFamily: "monospace", width: 80 }}>{l.name}</span>
            <span style={{ fontSize: 11, color: l.color, fontFamily: "monospace", fontWeight: 600 }}>{l.pct}%</span>
          </div>
        ))}
      </div>
    </div>
  );
}

const typeColors = { push: COLORS.accent, pr: COLORS.blue, issue: COLORS.red, star: "#FBBF24", release: COLORS.green };

export default function GitHubDashboard() {
  const [activeRepo, setActiveRepo] = useState(0);
  const [tick, setTick] = useState(0);

  useEffect(() => {
    const id = setInterval(() => setTick(t => t + 1), 2000);
    return () => clearInterval(id);
  }, []);

  const totalStars = REPOS.reduce((s, r) => s + r.stars, 0);
  const totalCommits = REPOS.reduce((s, r) => s + r.commits, 0);

  return (
    <div style={{
      minHeight: "100vh",
      background: COLORS.bg,
      fontFamily: "'Space Mono', 'Courier New', monospace",
      color: COLORS.text,
      padding: 0,
      position: "relative",
      overflow: "hidden",
    }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@600;800&display=swap');
        * { box-sizing: border-box; margin: 0; padding: 0; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: ${COLORS.surface}; }
        ::-webkit-scrollbar-thumb { background: ${COLORS.border}; border-radius: 2px; }
        .repo-row:hover { background: #161E28 !important; cursor: pointer; }
        .feed-item:hover { background: #161E28 !important; }
        .stat-card:hover { border-color: ${COLORS.accent}44 !important; transform: translateY(-1px); }
        .stat-card { transition: all 0.2s; }
        @keyframes pulse { 0%,100% { opacity: 1; } 50% { opacity: 0.4; } }
        @keyframes scanline {
          0% { transform: translateY(-100%); }
          100% { transform: translateY(100vh); }
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
        .fade-in { animation: fadeIn 0.5s ease both; }
      `}</style>

      {/* Scanline effect */}
      <div style={{
        position: "fixed", top: 0, left: 0, right: 0, height: 2,
        background: `linear-gradient(transparent, ${COLORS.accent}22, transparent)`,
        animation: "scanline 8s linear infinite",
        pointerEvents: "none", zIndex: 10,
      }} />

      {/* Grid bg */}
      <div style={{
        position: "fixed", inset: 0, zIndex: 0,
        backgroundImage: `linear-gradient(${COLORS.border}18 1px, transparent 1px), linear-gradient(90deg, ${COLORS.border}18 1px, transparent 1px)`,
        backgroundSize: "40px 40px",
        pointerEvents: "none",
      }} />

      <div style={{ position: "relative", zIndex: 1, maxWidth: 1300, margin: "0 auto", padding: "24px 24px" }}>

        {/* HEADER */}
        <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 28 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 16 }}>
            <div style={{
              width: 44, height: 44, borderRadius: 12,
              background: `linear-gradient(135deg, ${COLORS.accent}, #E08000)`,
              display: "flex", alignItems: "center", justifyContent: "center",
              fontSize: 22, boxShadow: `0 0 20px ${COLORS.accent}55`,
            }}>⬡</div>
            <div>
              <div style={{ fontFamily: "'Syne', sans-serif", fontSize: 22, fontWeight: 800, letterSpacing: "-0.5px", color: "#fff" }}>
                ghost<span style={{ color: COLORS.accent }}>hub</span>
              </div>
              <div style={{ fontSize: 10, color: COLORS.textDim, letterSpacing: 2, marginTop: 1 }}>MISSION CONTROL</div>
            </div>
          </div>

          <div style={{ display: "flex", alignItems: "center", gap: 20 }}>
            <div style={{ display: "flex", alignItems: "center", gap: 6 }}>
              <div style={{ width: 7, height: 7, borderRadius: "50%", background: COLORS.green, boxShadow: `0 0 8px ${COLORS.green}`, animation: "pulse 2s infinite" }} />
              <span style={{ fontSize: 11, color: COLORS.textDim }}>LIVE</span>
            </div>
            <div style={{
              background: COLORS.card, border: `1px solid ${COLORS.border}`,
              borderRadius: 8, padding: "6px 14px", display: "flex", alignItems: "center", gap: 10
            }}>
              <div style={{ width: 28, height: 28, borderRadius: 8, background: `linear-gradient(135deg, ${COLORS.purple}, ${COLORS.blue})`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 14 }}>👤</div>
              <div>
                <div style={{ fontSize: 12, color: COLORS.text, fontWeight: 700 }}>@ghostdev</div>
                <div style={{ fontSize: 10, color: COLORS.textDim }}>Pro · 5 repos</div>
              </div>
            </div>
          </div>
        </div>

        {/* STAT CARDS */}
        <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 14, marginBottom: 20 }}>
          {[
            { label: "TOTAL STARS", value: totalStars.toLocaleString(), delta: "+124", icon: "★", color: "#FBBF24" },
            { label: "COMMITS (30d)", value: "847", delta: "+23%", icon: "↑", color: COLORS.green },
            { label: "OPEN ISSUES", value: "114", delta: "-8", icon: "⚠", color: COLORS.red },
            { label: "PULL REQUESTS", value: "22", delta: "+5", icon: "⤷", color: COLORS.blue },
          ].map((s, i) => (
            <div key={i} className="stat-card fade-in" style={{
              background: COLORS.card,
              border: `1px solid ${COLORS.border}`,
              borderRadius: 12, padding: "18px 20px",
              animationDelay: `${i * 0.08}s`,
            }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                <div>
                  <div style={{ fontSize: 9, color: COLORS.textDim, letterSpacing: 2, marginBottom: 8 }}>{s.label}</div>
                  <div style={{ fontSize: 28, fontWeight: 700, color: "#fff", fontFamily: "'Syne', sans-serif", lineHeight: 1 }}>{s.value}</div>
                </div>
                <div style={{
                  width: 36, height: 36, borderRadius: 9,
                  background: `${s.color}18`,
                  border: `1px solid ${s.color}33`,
                  display: "flex", alignItems: "center", justifyContent: "center",
                  fontSize: 16, color: s.color,
                }}>{s.icon}</div>
              </div>
              <div style={{ marginTop: 12, fontSize: 11, color: s.delta.startsWith("+") ? COLORS.green : s.delta.startsWith("-") && s.label === "OPEN ISSUES" ? COLORS.green : COLORS.red }}>
                {s.delta} <span style={{ color: COLORS.textDim }}>this month</span>
              </div>
            </div>
          ))}
        </div>

        {/* MAIN GRID */}
        <div style={{ display: "grid", gridTemplateColumns: "1fr 320px", gap: 14 }}>

          {/* LEFT */}
          <div style={{ display: "flex", flexDirection: "column", gap: 14 }}>

            {/* REPOS TABLE */}
            <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, overflow: "hidden" }} className="fade-in">
              <div style={{ padding: "16px 20px", borderBottom: `1px solid ${COLORS.border}`, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <span style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim }}>REPOSITORIES</span>
                <span style={{ fontSize: 10, color: COLORS.accent, border: `1px solid ${COLORS.accent}44`, borderRadius: 20, padding: "2px 10px" }}>5 active</span>
              </div>
              <div>
                {REPOS.map((repo, i) => (
                  <div key={i} className="repo-row" onClick={() => setActiveRepo(i)} style={{
                    padding: "14px 20px",
                    borderBottom: i < REPOS.length - 1 ? `1px solid ${COLORS.border}` : "none",
                    display: "grid", gridTemplateColumns: "1fr 100px 80px 80px 80px 130px",
                    alignItems: "center", gap: 12,
                    background: activeRepo === i ? "#161E28" : "transparent",
                    borderLeft: activeRepo === i ? `2px solid ${repo.color}` : "2px solid transparent",
                    transition: "all 0.15s",
                  }}>
                    <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                      <div style={{ width: 8, height: 8, borderRadius: "50%", background: repo.color, boxShadow: `0 0 8px ${repo.color}` }} />
                      <div>
                        <div style={{ fontSize: 13, fontWeight: 700, color: "#fff" }}>{repo.name}</div>
                        <div style={{ fontSize: 10, color: COLORS.textDim, marginTop: 1 }}>{repo.commits.toLocaleString()} commits</div>
                      </div>
                    </div>
                    <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                      <div style={{ width: 7, height: 7, borderRadius: "50%", background: repo.color, opacity: 0.8 }} />
                      <span style={{ fontSize: 11, color: COLORS.textDim }}>{repo.lang}</span>
                    </div>
                    <div style={{ fontSize: 12, color: "#FBBF24" }}>★ {repo.stars.toLocaleString()}</div>
                    <div style={{ fontSize: 12, color: COLORS.textDim }}>⑂ {repo.forks}</div>
                    <div style={{ fontSize: 12, color: repo.issues > 20 ? COLORS.red : COLORS.textDim }}>⚠ {repo.issues}</div>
                    <SparkLine data={repo.trend} color={repo.color} />
                  </div>
                ))}
              </div>
            </div>

            {/* ACTIVITY BAR + CONTRIB */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14 }}>
              <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, padding: 20 }} className="fade-in">
                <div style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim, marginBottom: 16 }}>WEEKLY ACTIVITY</div>
                <BarChart data={ACTIVITY} />
                <div style={{ display: "flex", gap: 14, marginTop: 12 }}>
                  {[["Commits", COLORS.accent], ["PRs", COLORS.blue], ["Reviews", COLORS.purple]].map(([l, c]) => (
                    <div key={l} style={{ display: "flex", alignItems: "center", gap: 5 }}>
                      <div style={{ width: 8, height: 8, borderRadius: 2, background: c }} />
                      <span style={{ fontSize: 10, color: COLORS.textDim }}>{l}</span>
                    </div>
                  ))}
                </div>
              </div>

              <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, padding: 20 }} className="fade-in">
                <div style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim, marginBottom: 16 }}>CONTRIBUTIONS</div>
                <ContribGrid />
                <div style={{ marginTop: 10, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                  <span style={{ fontSize: 10, color: COLORS.textDim }}>826 contributions this year</span>
                  <div style={{ display: "flex", gap: 3, alignItems: "center" }}>
                    {["#1C2733", "#1B4332", "#276749", "#2ECC71", "#52E090"].map((c, i) => (
                      <div key={i} style={{ width: 8, height: 8, borderRadius: 2, background: c }} />
                    ))}
                  </div>
                </div>
              </div>
            </div>
          </div>

          {/* RIGHT SIDEBAR */}
          <div style={{ display: "flex", flexDirection: "column", gap: 14 }}>

            {/* LANGUAGE DONUT */}
            <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, padding: 20 }} className="fade-in">
              <div style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim, marginBottom: 16 }}>LANGUAGES</div>
              <DonutChart langs={LANGS} />
            </div>

            {/* LIVE FEED */}
            <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, overflow: "hidden", flex: 1 }} className="fade-in">
              <div style={{ padding: "14px 18px", borderBottom: `1px solid ${COLORS.border}`, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <span style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim }}>LIVE FEED</span>
                <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
                  <div style={{ width: 6, height: 6, borderRadius: "50%", background: COLORS.green, animation: "pulse 1.5s infinite" }} />
                  <span style={{ fontSize: 9, color: COLORS.green }}>STREAMING</span>
                </div>
              </div>
              <div>
                {FEED.map((item, i) => (
                  <div key={i} className="feed-item" style={{
                    padding: "12px 18px",
                    borderBottom: i < FEED.length - 1 ? `1px solid ${COLORS.border}44` : "none",
                    display: "flex", gap: 10, alignItems: "flex-start",
                    transition: "background 0.15s",
                  }}>
                    <div style={{ fontSize: 18, lineHeight: 1, marginTop: 1 }}>{item.avatar}</div>
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ display: "flex", alignItems: "center", gap: 5, marginBottom: 3 }}>
                        <span style={{ fontSize: 9, fontWeight: 700, color: typeColors[item.type], letterSpacing: 1, textTransform: "uppercase", background: `${typeColors[item.type]}18`, padding: "1px 6px", borderRadius: 3 }}>{item.type}</span>
                        <span style={{ fontSize: 10, color: COLORS.accent }}>/{item.repo}</span>
                      </div>
                      <div style={{ fontSize: 11, color: COLORS.text, lineHeight: 1.4, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>{item.msg}</div>
                      <div style={{ fontSize: 9, color: COLORS.textDim, marginTop: 4 }}>{item.time}</div>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            {/* QUICK STATS */}
            <div style={{ background: COLORS.card, border: `1px solid ${COLORS.border}`, borderRadius: 14, padding: 18 }} className="fade-in">
              <div style={{ fontSize: 11, letterSpacing: 2, color: COLORS.textDim, marginBottom: 14 }}>AT A GLANCE</div>
              {[
                ["Avg Stars / Repo", "2,505", COLORS.accent],
                ["PR Merge Rate", "91%", COLORS.green],
                ["Avg Issue Close", "4.2d", COLORS.blue],
                ["Code Coverage", "87%", COLORS.purple],
              ].map(([label, val, color], i) => (
                <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: i < 3 ? 10 : 0 }}>
                  <span style={{ fontSize: 11, color: COLORS.textDim }}>{label}</span>
                  <span style={{ fontSize: 13, fontWeight: 700, color, fontFamily: "'Syne', sans-serif" }}>{val}</span>
                </div>
              ))}
            </div>
          </div>
        </div>

        {/* FOOTER */}
        <div style={{ marginTop: 16, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
          <span style={{ fontSize: 10, color: COLORS.muted }}>ghosthub · mission control v2.4</span>
          <span style={{ fontSize: 10, color: COLORS.muted, fontFamily: "monospace" }}>
            LAST SYNC: {new Date().toLocaleTimeString()}
          </span>
        </div>
      </div>
    </div>
  );
}
