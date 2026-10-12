export const DEFAULT_RATES = { Comercial: 2500000, Oficinas: 2000000, Residencial: 1800000 };
export const DEFAULT_HEIGHTS = { Comercial: 3.25, Oficinas: 3.0, Residencial: 2.8 };
export const COLORS = { Comercial: 0xc5a66a, Oficinas: 0x7296a8, Residencial: 0x55a58c };

export function calculateTotalHeight(levels) {
  return levels.reduce((sum, level) => sum + validPositive(level.height), 0);
}
export function calculateConstructionArea(levels, lotArea) {
  return levels.length * Math.max(0, Number(lotArea) || 0);
}
export function calculateVolume(lotArea, levels) {
  return Math.max(0, Number(lotArea) || 0) * calculateTotalHeight(levels);
}
export function calculateCoreArea(levelCount, b, lotArea) {
  const x = Math.max(0, Number(levelCount) || 0);
  const raw = -x * x + Math.max(0, Number(b) || 0) * x;
  // El modelo cuadrático es didáctico; el área no puede ser negativa ni superar el 30% de la planta por nivel.
  return Math.min(Math.max(0, raw), Math.max(0, Number(lotArea) || 0) * 0.3);
}
export function calculateUsableArea(builtArea, coreArea) {
  return Math.max(0, (Number(builtArea) || 0) - (Number(coreArea) || 0));
}
export function calculateLevelCost(level, lotArea, rates) {
  return Math.max(0, Number(lotArea) || 0) * Math.max(0, Number(rates[level.use]) || 0);
}
export function calculateTotalCost(levels, lotArea, rates, fixedCosts) {
  return levels.reduce((sum, level) => sum + calculateLevelCost(level, lotArea, rates), 0) + Math.max(0, Number(fixedCosts) || 0);
}
export function validPositive(value) {
  const n = Number(value);
  return Number.isFinite(n) && n > 0 ? n : 0;
}
export function formatNumber(value, digits = 2) {
  return (Number.isFinite(value) ? value : 0).toLocaleString('es-CO', { minimumFractionDigits: digits, maximumFractionDigits: digits });
}
export function formatMoney(value) {
  return '$ ' + Math.round(Number.isFinite(value) ? value : 0).toLocaleString('es-CO');
}
