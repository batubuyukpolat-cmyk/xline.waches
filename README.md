import React, { useState, useEffect } from "react";

// Ürün Kartı
const ProductCard = ({ product = {}, saveProducts = () => {} }) => {
  const safeProduct = {
    name: product.name || "Ürün",
    price: Number(product.price) || 0,
    image: product.image || "https://via.placeholder.com/250",
    sold: Number(product.sold) || 0,
    comments: Array.isArray(product.comments) ? product.comments : []
  };

  const [comments, setComments] = useState(safeProduct.comments);
  const [newComment, setNewComment] = useState("");

  const handleBuy = () => {
    const updated = { ...safeProduct, sold: safeProduct.sold + 1 };
    let allProducts = [];
    try {
      const raw = localStorage.getItem("xline_products");
      allProducts = raw ? JSON.parse(raw) : [];
      if (!Array.isArray(allProducts)) allProducts = [];
    } catch {
      allProducts = [];
    }
    const updatedProducts = allProducts.map(p => p.name === updated.name ? updated : p);
    saveProducts(updatedProducts);
    alert("Papara ile ödeme simülasyonu: Ödeme başarılı!");
  };

  const addComment = () => {
    if (!newComment.trim()) return;
    const updatedComments = [...comments, newComment.trim()];
    setComments(updatedComments);
    setNewComment("");
    const updatedProduct = { ...safeProduct, comments: updatedComments };
    let allProducts = [];
    try {
      const raw = localStorage.getItem("xline_products");
      allProducts = raw ? JSON.parse(raw) : [];
      if (!Array.isArray(allProducts)) allProducts = [];
    } catch {
      allProducts = [];
    }
    const updatedProducts = allProducts.map(p => p.name === updatedProduct.name ? updatedProduct : p);
    saveProducts(updatedProducts);
  };

  return (
    <div style={{ border: "1px solid gray", padding: "10px", width: "250px", borderRadius: "10px" }}>
      <img src={safeProduct.image} alt={safeProduct.name} style={{ width: "100%", borderRadius: "10px" }} />
      <h3>{safeProduct.name}</h3>
      <p>Fiyat: {safeProduct.price} TL</p>
      <p>Satış: {safeProduct.sold}</p>
      <button onClick={handleBuy} style={{ marginBottom: "10px" }}>Satın Al</button>

      <div>
        <h4>Yorumlar</h4>
        {comments.length === 0 ? <p>Henüz yorum yok</p> : comments.map((c, i) => <p key={i}>- {c}</p>)}
        <input
          type="text"
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          placeholder="Yorum yaz..."
        />
        <button onClick={addComment}>Ekle</button>
      </div>
    </div>
  );
};

// Admin Paneli
const AdminPanel = ({ products = [], saveProducts = () => {} }) => {
  const [name, setName] = useState("");
  const [price, setPrice] = useState("");
  const [image, setImage] = useState("");

  const addProduct = () => {
    if (!name.trim() || !price.trim() || !image.trim()) return;
    const newProduct = {
      name: name.trim(),
      price: Number(price) || 0,
      image: image.trim(),
      sold: 0,
      comments: []
    };
    saveProducts([...products, newProduct]);
    setName(""); setPrice(""); setImage("");
  };

  return (
    <div style={{ padding: "20px" }}>
      <h2>Admin Panel</h2>
      <input placeholder="Ürün adı" value={name} onChange={(e) => setName(e.target.value)} />
      <input placeholder="Fiyat" type="number" value={price} onChange={(e) => setPrice(e.target.value)} />
      <input placeholder="Resim URL" value={image} onChange={(e) => setImage(e.target.value)} />
      <button onClick={addProduct}>Ürün Ekle</button>

      <h3>Mevcut Ürünler</h3>
      {products.length === 0 ? <p>Henüz ürün yok</p> : products.map((p, i) => (
        <div key={i}>
          {p.name} - {p.price} TL - Satış: {p.sold}
        </div>
      ))}
    </div>
  );
};

// Ana App
const App = () => {
  const defaultProducts = [
    { name: "Xline Classic", price: 1200, image: "https://via.placeholder.com/250", sold: 0, comments: [] },
    { name: "Xline Sport", price: 1500, image: "https://via.placeholder.com/250", sold: 0, comments: [] }
  ];

  const [products, setProducts] = useState([]);
  const [adminMode, setAdminMode] = useState(false);

  useEffect(() => {
    let saved = [];
    try {
      const raw = localStorage.getItem("xline_products");
      saved = raw ? JSON.parse(raw) : [];
      if (!Array.isArray(saved)) saved = [];
    } catch {
      saved = [];
    }
    setProducts(saved.length ? saved : defaultProducts);
  }, []);

  const saveProducts = (newProducts) => {
    setProducts(newProducts);
    try {
      localStorage.setItem("xline_products", JSON.stringify(newProducts));
    } catch {}
  };

  return (
    <div style={{ fontFamily: "Arial, sans-serif" }}>
      <header style={{ textAlign: "center", padding: "20px", background: "#f0f0f0" }}>
        <h1>Xline Watches</h1>
        <button onClick={() => setAdminMode(!adminMode)}>
          {adminMode ? "Çıkış" : "Admin Girişi"}
        </button>
      </header>

      {adminMode ? (
        <AdminPanel products={products} saveProducts={saveProducts} />
      ) : (
        <div style={{ display: "flex", flexWrap: "wrap", gap: "20px", justifyContent: "center", padding: "20px" }}>
          {products.map((p, i) => <ProductCard key={i} product={p} saveProducts={saveProducts} />)}
        </div>
      )}
    </div>
  );
};

export default App;
