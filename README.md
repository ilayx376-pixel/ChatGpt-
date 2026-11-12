# ChatGpt-/*- Single-file React component (App.jsx)
LCD-style version: Uses 'Seven Segment' font for real meter reading look + font & decimal adjustment.
*/

import React, { useRef, useState, useEffect } from "react";
import html2canvas from "html2canvas";

export default function App() {
  const [imageSrc, setImageSrc] = useState(null);
  const [text, setText] = useState("");
  const [fontSize, setFontSize] = useState(48);
  const [color, setColor] = useState("#00ff00");
  const [fontFamily, setFontFamily] = useState("Seven Segment, monospace");
  const [dragging, setDragging] = useState(false);
  const [pos, setPos] = useState({ x: 50, y: 50 });
  const [offset, setOffset] = useState({ x: 0, y: 0 });
  const [zoom, setZoom] = useState(1);
  const canvasRef = useRef(null);
  const textRef = useRef(null);

  useEffect(() => {
    const id = 'seven-segment-font';
    if (!document.getElementById(id)) {
      const link = document.createElement('link');
      link.id = id;
      link.rel = 'stylesheet';
      link.href = 'https://fonts.cdnfonts.com/css/seven-segment';
      document.head.appendChild(link);
    }
  }, []);

  function handleImageUpload(e) {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = () => setImageSrc(reader.result);
    reader.readAsDataURL(file);
  }

  async function handleDownload() {
    if (!canvasRef.current) return;
    const canvasNode = canvasRef.current;
    const blobCanvas = await html2canvas(canvasNode, { useCORS: true, scale: 2 });
    const link = document.createElement("a");
    link.download = "pro-meter-lcd.png";
    link.href = blobCanvas.toDataURL("image/png");
    link.click();
  }

  function onPointerDown(e) {
    e.preventDefault();
    setDragging(true);
    const rect = e.currentTarget.parentNode.getBoundingClientRect();
    const startX = e.clientX || (e.touches && e.touches[0].clientX);
    const startY = e.clientY || (e.touches && e.touches[0].clientY);
    const ele = textRef.current.getBoundingClientRect();
    setOffset({ x: startX - ele.left, y: startY - ele.top, rectLeft: rect.left, rectTop: rect.top });
  }

  function onPointerMove(e) {
    if (!dragging) return;
    const rect = canvasRef.current.getBoundingClientRect();
    const clientX = e.clientX || (e.touches && e.touches[0].clientX);
    const clientY = e.clientY || (e.touches && e.touches[0].clientY);
    const x = ((clientX - rect.left - offset.x + (textRef.current ? textRef.current.offsetWidth / 2 : 0)) / rect.width) * 100;
    const y = ((clientY - rect.top - offset.y + (textRef.current ? textRef.current.offsetHeight / 2 : 0)) / rect.height) * 100;
    setPos({ x: Math.max(2, Math.min(98, x)), y: Math.max(2, Math.min(98, y)) });
  }

  function onPointerUp() {
    setDragging(false);
  }

  useEffect(() => {
    window.addEventListener("pointerup", onPointerUp);
    window.addEventListener("touchend", onPointerUp);
    return () => {
      window.removeEventListener("pointerup", onPointerUp);
      window.removeEventListener("touchend", onPointerUp);
    };
  }, []);

  function handleTextChange(e) {
    const value = e.target.value.replace(/[^0-9.]/g, ""); // Allow digits and decimal point
    setText(value);
  }

  return (
    <div style={{ minHeight: "100vh", background: "#0f1724", color: "#fff", fontFamily: 'sans-serif', padding: 20 }}>
      <div style={{ maxWidth: 960, margin: "0 auto", display: "flex", gap: 20, flexWrap: "wrap" }}>
        <div style={{ flex: "1 1 340px", minWidth: 300 }}>
          <h1 style={{ color: '#39f', marginBottom: 12 }}>Meter Display Editor</h1>

          <div style={{ background: '#0b1320', padding: 14, borderRadius: 12, boxShadow: '0 6px 20px rgba(0,0,0,0.6)' }}>
            <label style={{ display: 'block', marginBottom: 8, fontWeight: 600 }}>Upload Image</label>
            <input type="file" accept="image/*" onChange={handleImageUpload} style={{ width: '100%', padding: 8, borderRadius: 8 }} />

            <div style={{ height: 12 }} />

            <label style={{ display: 'block', marginBottom: 6, fontWeight: 600 }}>Enter Meter Reading</label>
            <input value={text} onChange={handleTextChange} placeholder="Enter numbers or decimal (e.g. 123.45)" style={{ width: '100%', padding: 8, borderRadius: 8, fontFamily: fontFamily, fontSize: 20 }} />

            <div style={{ marginTop: 12 }}>
              <label style={{ fontSize: 12 }}>Font Family</label>
              <select value={fontFamily} onChange={(e) => setFontFamily(e.target.value)} style={{ width: '100%', padding: 8, borderRadius: 8 }}>
                <option value="Seven Segment, monospace">Seven Segment (LCD)</option>
                <option value="monospace">Monospace</option>
                <option value="Arial, sans-serif">Arial</option>
                <option value="Courier New, monospace">Courier New</option>
                <option value="Orbitron, sans-serif">Orbitron</option>
              </select>
            </div>

            <div style={{ marginTop: 12 }}>
              <label style={{ fontSize: 12 }}>Size: {fontSize}px</label>
              <input type="range" min={12} max={120} value={fontSize} onChange={(e) => setFontSize(parseInt(e.target.value))} style={{ width: '100%' }} />
            </div>

            <div style={{ marginTop: 12 }}>
              <label style={{ fontSize: 12 }}>Color</label>
              <input type="color" value={color} onChange={(e) => setColor(e.target.value)} style={{ width: 48, height: 36, borderRadius: 6, border: 'none' }} />
            </div>

            <div style={{ marginTop: 12 }}>
              <label style={{ fontSize: 12 }}>Zoom: {Math.round(zoom * 100)}%</label>
              <input type="range" min={0.5} max={2} step={0.05} value={zoom} onChange={(e) => setZoom(parseFloat(e.target.value))} style={{ width: '100%' }} />
            </div>

            <button onClick={handleDownload} style={{ width: '100%', marginTop: 12, padding: '10px 12px', background: 'linear-gradient(90deg,#6a11cb,#2575fc)', border: 'none', color: '#fff', borderRadius: 999, fontWeight: 700 }}>⬇️ Save Image</button>
          </div>
        </div>

        <div style={{ flex: '1 1 480px', minWidth: 320 }}>
          <div style={{ marginBottom: 8, fontWeight: 600 }}>Preview</div>
          <div ref={canvasRef} style={{ width: '100%', maxWidth: 480, height: 720, background: '#1a2430', borderRadius: 16, overflow: 'hidden', position: 'relative', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
            {imageSrc ? (
              <img src={imageSrc} alt="uploaded" style={{ width: '100%', height: '100%', objectFit: 'contain', transform: `scale(${zoom})`, transformOrigin: 'center center' }} />
            ) : (
              <div style={{ color: '#9fb0c9' }}>Upload an image to begin</div>
            )}

            <div
              ref={textRef}
              onMouseDown={onPointerDown}
              onTouchStart={onPointerDown}
              onMouseMove={onPointerMove}
              onTouchMove={onPointerMove}
              style={{
                position: 'absolute',
                left: `${pos.x}%`,
                top: `${pos.y}%`,
                transform: 'translate(-50%, -50%)',
                cursor: dragging ? 'grabbing' : 'grab',
                userSelect: 'none',
                fontFamily: fontFamily,
                fontSize: fontSize,
                color: color,
                textShadow: '0 2px 8px rgba(0,0,0,0.7)'
              }}
            >
              {text}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
